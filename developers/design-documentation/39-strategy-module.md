# 39 - Strategy Module

## Overview

The Strategy Module provides a webhook-based system for receiving trading signals from external platforms and executing orders through OpenAlgo. It features time-based controls, symbol mappings, automatic square-off, and rate-limited order queuing.

A platform must be chosen when a strategy is created, and the stored strategy name is `<platform>_<name>`. The platform values offered by the UI (`frontend/src/types/strategy.ts`) are `tradingview`, `amibroker`, `python`, `metatrader`, `excel` and `others`. Chartink has its own separate module, documented in [13 - Chartink Architecture](13-chartink-architecture.md).

## Architecture Diagram

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                        Strategy Module Architecture                          │
└──────────────────────────────────────────────────────────────────────────────┘

┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
│   TradingView   │  │   Amibroker     │  │  Custom client  │
│   Webhook       │  │   Webhook       │  │  or script      │
└────────┬────────┘  └────────┬────────┘  └────────┬────────┘
         │                    │                    │
         └────────────────────┼────────────────────┘
                              │
                              ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│                          Strategy Webhook Endpoint                           │
│                     POST /strategy/webhook/<webhook_id>                      │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐     │
│  │  1. Rate Limiting (100/min for webhooks)                             │    │
│  │  2. Validate webhook_id → Get strategy                               │    │
│  │  3. Check strategy enabled & time window                             │    │
│  │  4. Parse signal (action, symbol, quantity)                          │    │
│  │  5. Apply symbol mapping overrides                                   │    │
│  │  6. Queue order for execution                                        │    │
│  └─────────────────────────────────────────────────────────────────────┘     │
└──────────────────────────────────────────────────────────────────────────────┘
                                       │
                                       ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│                            Order Queueing System                             │
│                                                                              │
│  ┌──────────────────────┐        ┌──────────────────────┐                    │
│  │   Regular Queue      │        │   Smart Order Queue  │                    │
│  │   (placeorder)       │        │   (placesmartorder)  │                    │
│  │                      │        │                      │                    │
│  │   Rate: 10/sec       │        │   Rate: 1/sec        │                    │
│  │   (hard-coded)       │        │   (hard-coded)       │                    │
│  └──────────┬───────────┘        └──────────┬───────────┘                    │
│             │                               │                                │
│             └───────────────┬───────────────┘                                │
│                             │                                                │
│                             ▼                                                │
│                    ┌─────────────────┐                                       │
│                    │ Order Processor │                                       │
│                    │ (background)    │                                       │
│                    └────────┬────────┘                                       │
│                             │                                                │
└─────────────────────────────┼────────────────────────────────────────────────┘
                              │
                              ▼
                     ┌─────────────────┐
                     │ REST API        │
                     │ /api/v1/...     │
                     └─────────────────┘
```

## Strategy Configuration

### Database Schema

**Location:** `database/strategy_db.py`

```python
class Strategy(Base):
    __tablename__ = 'strategies'

    id = Column(Integer, primary_key=True)
    name = Column(String(255), nullable=False)
    webhook_id = Column(String(36), unique=True, nullable=False)   # UUID
    user_id = Column(String(255), nullable=False)                  # Owner
    platform = Column(String(50), nullable=False,
                      default='tradingview')                       # tradingview, chartink, etc
    is_active = Column(Boolean, default=True)                      # Active/inactive
    is_intraday = Column(Boolean, default=True)                    # Intraday or positional
    trading_mode = Column(String(10), nullable=False,
                          default='LONG')                          # LONG, SHORT, BOTH
    start_time = Column(String(5))                                 # HH:MM (09:15)
    end_time = Column(String(5))                                   # HH:MM (15:15)
    squareoff_time = Column(String(5))                             # HH:MM (15:25)
    created_at = Column(DateTime(timezone=True), server_default=func.now())
    updated_at = Column(DateTime(timezone=True), onupdate=func.now())

class StrategySymbolMapping(Base):
    __tablename__ = 'strategy_symbol_mappings'

    id = Column(Integer, primary_key=True)
    strategy_id = Column(Integer, ForeignKey('strategies.id'), nullable=False)
    symbol = Column(String(50), nullable=False)        # OpenAlgo symbol to trade
    exchange = Column(String(10), nullable=False)      # NSE, NFO, etc.
    quantity = Column(Integer, nullable=False)         # Order quantity
    product_type = Column(String(10), nullable=False)  # MIS/CNC/NRML
    created_at = Column(DateTime(timezone=True), server_default=func.now())
    updated_at = Column(DateTime(timezone=True), onupdate=func.now())
```

> **Note**: There is no `signal_symbol` column. The webhook's `symbol` value is matched directly against `StrategySymbolMapping.symbol`, so a signal must already use the OpenAlgo symbol name.

Webhook and user-strategy lookups are cached in `database/strategy_db.py` with two `TTLCache` instances: `_strategy_webhook_cache` (maxsize 5000, TTL 300 seconds) and `_user_strategies_cache` (maxsize 1000, TTL 600 seconds).

### Valid Exchanges and Products

`blueprints/strategy.py` restricts symbol mappings to these combinations, with `DEFAULT_EXCHANGE = "NSE"` and `DEFAULT_PRODUCT = "MIS"`:

| Exchange | Allowed Products |
|----------|------------------|
| NSE | MIS, CNC |
| BSE | MIS, CNC |
| NFO | MIS, NRML |
| CDS | MIS, NRML |
| BFO | MIS, NRML |
| BCD | MIS, NRML |
| MCX | MIS, NRML |
| NCDEX | MIS, NRML |

### Time Validation

```python
def validate_strategy_times(start_time, end_time, squareoff_time):
    """Validate strategy time settings"""

    # Market hours (9:15 AM to 3:30 PM)
    market_open = time(9, 15)
    market_close = time(15, 30)

    # Validations:
    # 1. All three fields are required
    # 2. Start time >= market_open (09:15)
    # 3. End time <= market_close (15:30)
    # 4. Square off time <= market_close (15:30)
    # 5. Start time < End time
    # 6. Square off time >= Start time and >= End time
```

Strategy names are validated separately by `validate_strategy_name()`: 3 to 50 characters, matching `^[A-Za-z0-9\s\-_]+$`. Unlike the Chartink module, no prefix is added to the name.

## Webhook Signal Format

The handler reads only `symbol`, `action` and `position_size` from the body. Required fields are `symbol` and `action`, plus `position_size` when the strategy's `trading_mode` is `BOTH`. Exchange, product and quantity are never taken from the signal: they come from the symbol mapping.

### LONG or SHORT mode

```json
{
    "symbol": "SBIN",
    "action": "BUY"
}
```

### BOTH mode

```json
{
    "symbol": "SBIN",
    "action": "BUY",
    "position_size": 100
}
```

### TradingView Format

```json
{
    "symbol": "{{ticker}}",
    "action": "{{strategy.order.action}}",
    "position_size": "{{strategy.position_size}}"
}
```

### Supported Actions

| Action | Description |
|--------|-------------|
| `BUY` | Long entry / Short exit |
| `SELL` | Long exit / Short entry |

Any action other than `BUY` or `SELL` is rejected with HTTP 400.

## Symbol Mapping

The mapping is looked up by exact match of the signal's `symbol` against `StrategySymbolMapping.symbol`. It supplies the exchange, product type and quantity. A signal for a symbol with no mapping is rejected with HTTP 400.

```
External Signal: "SBIN"
       │
       ▼
┌──────────────────────────────────────┐
│  Symbol Mapping Lookup               │
│  Exact match on symbol               │
│                                      │
│  "SBIN" → {                          │
│    symbol: "SBIN",                   │
│    exchange: "NSE",                  │
│    product_type: "MIS",              │
│    quantity: 50                      │
│  }                                   │
└──────────────────────────────────────┘
       │
       ▼
Place Order: NSE:SBIN, Qty: 50, Product: MIS
```

## Order Queuing System

### Dual Queue Architecture

```python
# Separate queues for different order types
regular_order_queue = queue.Queue()  # For placeorder (up to 10/sec)
smart_order_queue = queue.Queue()    # For placesmartorder (1/sec)

def process_orders():
    """Background task to process orders from both queues with rate limiting"""
    while True:
        # 1. Process smart orders first (1 per second)
        try:
            smart_order = smart_order_queue.get_nowait()
            if smart_order is None:  # Poison pill
                break
            get_httpx_client().post(
                f'{BASE_URL}/api/v1/placesmartorder', json=smart_order['payload']
            )
            time_module.sleep(1)  # Always wait 1 second after a smart order
            continue
        except queue.Empty:
            pass

        # 2. Process regular orders (up to 10 per second)
        now = time()
        while last_regular_orders and now - last_regular_orders[0] > 1:
            last_regular_orders.popleft()

        if len(last_regular_orders) < 10:
            try:
                regular_order = regular_order_queue.get_nowait()
                if regular_order is None:  # Poison pill
                    break
                response = get_httpx_client().post(
                    f'{BASE_URL}/api/v1/placeorder', json=regular_order['payload']
                )
                if response.is_success:
                    last_regular_orders.append(now)
            except queue.Empty:
                pass

        time_module.sleep(0.1)  # Prevent CPU spinning
```

Orders are posted with the shared httpx client (`utils/httpx_client.get_httpx_client()`), not `requests`. The processor thread is started lazily by `ensure_order_processor()` on the first `queue_order()` call, and an `atexit` handler drains any pending orders (30 second join timeout) by pushing a poison pill onto the regular queue.

### Rate Limiting

| Order Type | Rate Limit | Queue |
|------------|------------|-------|
| Regular Order | 10/second, enforced by a `deque(maxlen=10)` of timestamps | `regular_order_queue` |
| Smart Order | 1/second, enforced by a fixed 1 second sleep | `smart_order_queue` |

These are hard-coded in `blueprints/strategy.py`. They are not read from `ORDER_RATE_LIMIT` or `SMART_ORDER_RATE_LIMIT`.

## Automatic Square-Off

### APScheduler Integration

```python
scheduler = BackgroundScheduler(
    timezone=pytz.timezone('Asia/Kolkata'),
    job_defaults={
        'coalesce': True,
        'misfire_grace_time': 300,
        'max_instances': 1
    }
)

def schedule_squareoff(strategy_id):
    """Schedule squareoff for intraday strategy"""
    strategy = get_strategy(strategy_id)
    hours, minutes = map(int, strategy.squareoff_time.split(':'))

    scheduler.add_job(
        squareoff_positions,
        'cron',
        hour=hours,
        minute=minutes,
        args=[strategy_id],
        id=f'squareoff_{strategy_id}',
        timezone=pytz.timezone('Asia/Kolkata')
    )
```

### Square-Off Logic

```python
def squareoff_positions(strategy_id):
    """Square off all positions for intraday strategy"""
    strategy = get_strategy(strategy_id)
    mappings = get_symbol_mappings(strategy_id)

    for mapping in mappings:
        payload = {
            'apikey': api_key,
            'symbol': mapping.symbol,
            'exchange': mapping.exchange,
            'product': mapping.product_type,
            'strategy': strategy.name,
            'action': 'SELL',   # Direction does not matter for closing
            'pricetype': 'MARKET',
            'quantity': '0',
            'position_size': '0',  # Closes position
            'price': '0',
            'trigger_price': '0',
            'disclosed_quantity': '0',
        }
        queue_order('placesmartorder', payload)
```

## API Endpoints

Blueprint: `strategy_bp`, `url_prefix="/strategy"`.

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/strategy/` | GET | List all strategies |
| `/strategy/new` | GET/POST | Create new strategy |
| `/strategy/<strategy_id>` | GET | View strategy details |
| `/strategy/toggle/<strategy_id>` | POST | Enable/disable strategy |
| `/strategy/<strategy_id>/delete` | POST | Delete strategy |
| `/strategy/<strategy_id>/configure` | GET/POST | Manage symbol mappings |
| `/strategy/<strategy_id>/symbol/<mapping_id>/delete` | POST | Delete one symbol mapping |
| `/strategy/search` | GET | Symbol search helper |
| `/strategy/api/strategies` | GET | List strategies as JSON |
| `/strategy/api/strategy/<strategy_id>` | GET | Strategy details as JSON |
| `/strategy/api/strategy` | POST | Create strategy as JSON |
| `/strategy/api/strategy/<strategy_id>/toggle` | POST | Toggle strategy as JSON |
| `/strategy/webhook/<webhook_id>` | POST | Receive trading signal |

There is no edit route. Rate limits actually applied: `WEBHOOK_RATE_LIMIT` on `/webhook/<webhook_id>`, and `STRATEGY_RATE_LIMIT` on `/new`, `/<strategy_id>/delete`, `/<strategy_id>/configure`, `/<strategy_id>/symbol/<mapping_id>/delete` and `/api/strategy`. The other routes carry no `@limiter.limit` decorator.

## Trading Modes

| Mode | Entry Action | Exit Action | Order Endpoint |
|------|--------------|-------------|----------------|
| `LONG` | BUY | SELL | BUY goes to `placeorder`, SELL goes to `placesmartorder` with `position_size=0` |
| `SHORT` | SELL | BUY | SELL goes to `placeorder`, BUY goes to `placesmartorder` with `position_size=0` |
| `BOTH` | BUY or SELL with `position_size` | Either action with `position_size=0` | Always `placesmartorder`, passing the signal's `position_size` through |

In `BOTH` mode a `BUY` requires `position_size >= 0` and a `SELL` requires `position_size <= 0`. In `LONG` and `SHORT` mode, an entry order uses `abs(position_size)` as the quantity when a non-zero `position_size` is supplied, otherwise the mapping's quantity.

### Time Window Enforcement

Time checks apply only when `is_intraday` is true. Entry orders are rejected before `start_time` and after `end_time`. Exit orders are rejected before `start_time` and after `squareoff_time`.

## Strategy Time Window

```
Market Hours: 09:15 ─────────────────────────────────────── 15:30
                    │                                      │
Strategy Window:    │  start_time ─────── end_time        │
                    │      │                  │            │
                    │      └──────────────────┘            │
                    │             ▲                        │
                    │     Signals accepted                 │
                    │                                      │
Square-off:         │                              squareoff_time
                    │                                      │
                    │                                    ──┼──
                    │                              Close all MIS
```

## Configuration

### Environment Variables

Defaults as written in `blueprints/strategy.py`:

```bash
WEBHOOK_RATE_LIMIT="100 per minute"
STRATEGY_RATE_LIMIT="200 per minute"
HOST_SERVER="http://127.0.0.1:5000"  # Base URL for internal API calls
```

## Key Files Reference

| File | Purpose |
|------|---------|
| `blueprints/strategy.py` | Strategy blueprint and webhook handler |
| `database/strategy_db.py` | Strategy database models |
| `frontend/src/pages/strategy/StrategyIndex.tsx` | Strategy list |
| `frontend/src/pages/strategy/NewStrategy.tsx` | Strategy creation |
| `frontend/src/pages/strategy/ViewStrategy.tsx` | Strategy details |
| `frontend/src/pages/strategy/ConfigureSymbols.tsx` | Symbol mappings |
