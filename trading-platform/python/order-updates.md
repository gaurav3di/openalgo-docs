# Order Updates (Real-Time)

Stream real-time order status changes — fills, partial fills, rejections and
cancellations — directly into your Python strategy through the OpenAlgo
WebSocket feed. This is an **account-level** stream: unlike LTP/Quote/Depth
there are no instruments to subscribe; one call covers every order in the
connected broker account (and sandbox orders in analyze mode).

***

### **Methods**

| Method                              | Purpose                                              |
| ----------------------------------- | ---------------------------------------------------- |
| `subscribe_orders(on_order_update)` | Start the stream; optional callback per update       |
| `unsubscribe_orders()`              | Stop the stream                                      |
| `get_orders()`                      | Cached updates received so far (most recent first)   |

The subscription survives reconnects — the SDK's auto-reconnect replays it
after re-authentication, exactly like market-data subscriptions.

***

### **Usage**

```python
from openalgo import api
import time

client = api(
    api_key="your_api_key_here",
    host="http://127.0.0.1:5000",
    ws_url="ws://127.0.0.1:8765"
)

def on_order_update(update):
    print(
        f"{update['orderid']} | {update['symbol']} | {update['order_status']} "
        f"| filled {update['filled_quantity']}/{update['quantity']} "
        f"@ {update['average_price']}"
    )

client.connect()
client.subscribe_orders(on_order_update=on_order_update)

# ... place orders from anywhere (API, web UI, mobile app) ...
time.sleep(60)

client.unsubscribe_orders()
client.disconnect()
```

***

### **Update Format**

Each callback receives OpenAlgo's common order format — the same constants
used by the REST order APIs:

```json
{
  "type": "order_update",
  "user_id": "openalgo-user",
  "mode": "live",
  "broker": "upstox",
  "orderid": "240221025997024",
  "symbol": "RELIANCE",
  "exchange": "NSE",
  "action": "BUY",
  "quantity": 10,
  "price": 1424.0,
  "trigger_price": 0,
  "pricetype": "LIMIT",
  "product": "MIS",
  "order_status": "complete",
  "filled_quantity": 10,
  "pending_quantity": 0,
  "average_price": 1423.85,
  "rejection_reason": ""
}
```

| Field              | Values                                                          |
| ------------------ | --------------------------------------------------------------- |
| `mode`             | `live` (broker feed) or `analyze` (sandbox engine)               |
| `action`           | `BUY` / `SELL`                                                   |
| `pricetype`        | `MARKET` / `LIMIT` / `SL` / `SL-M`                               |
| `product`          | `CNC` / `NRML` / `MIS`                                           |
| `order_status`     | `open` / `trigger pending` / `complete` / `rejected` / `cancelled` (+ broker extras) |
| `rejection_reason` | Populated when `order_status` is `rejected`                      |

***

### **Notes**

* Sandbox (analyze mode) fires the same updates for order placements
  (open), engine fills, rejections, and cancellations — test end-to-end without a live broker.
* Broker coverage: Zerodha, Dhan, Fyers, Upstox, AliceBlue, Definedge,
  IndMoney, Angel One, Nubra, and Arrow stream natively; Groww uses
  server-side orderbook polling as a fallback.
* With `verbose=2`, each update is also printed with the `[ORDER]` log tag.
