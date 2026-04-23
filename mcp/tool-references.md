# Tool References

## OpenAlgo MCP — Tool Reference & Prompt Examples

Companion reference to the main MCP setup guide. Once the MCP server is wired into Claude Desktop, Cursor, Windsurf, Antigravity, or any other MCP-capable client, you can ask for these operations in plain English — the client decides which tool to call.

All **40 tools** shipped by the server are listed below with:

* What the tool does
* Key parameters (required / optional)
* Example prompts you can paste directly into Claude / Cursor / Antigravity / Windsurf

### Conventions

* **Default strategy tag**: `python mcp` — every MCP-triggered order is tagged so you can filter MCP activity in OpenAlgo logs and the Analyzer. Override by saying _"use strategy 'my scalper'"_ in the prompt.
* **Product type defaults**: `MIS` for equity. Use `NRML` for F\&O carry; `CNC` for delivery.
* **Exchange codes**: `NSE`, `BSE`, `NFO`, `BFO`, `CDS`, `BCD`, `MCX` + `NSE_INDEX` / `BSE_INDEX` for index values.
* **Lot size**: never hardcoded. The model will call `get_option_symbol` / `get_option_chain` / `get_symbol_info` to read the live `lotsize` from the broker master contract, then compute `quantity = lots × lotsize` for you.

***

### 📦 Order Management

#### `place_order`

Place a single market / limit / stop-loss order.

| Param                                          | Required | Notes                                             |
| ---------------------------------------------- | -------- | ------------------------------------------------- |
| `symbol`, `quantity`, `action`                 | Yes      | —                                                 |
| `exchange`                                     | No       | Default `NSE`                                     |
| `price_type`                                   | No       | `MARKET`, `LIMIT`, `SL`, `SL-M`. Default `MARKET` |
| `product`                                      | No       | `CNC`, `NRML`, `MIS`. Default `MIS`               |
| `strategy`                                     | No       | Default `python mcp`                              |
| `price`, `trigger_price`, `disclosed_quantity` | No       | Use as applicable                                 |

**Prompts:**

* _"Place a market buy for 100 shares of RELIANCE on NSE, intraday"_
* _"Buy 50 INFY at limit 1550, delivery product"_
* _"Sell 25 SBIN with a stop-loss at 765 and trigger 766"_

***

#### `place_smart_order`

Auto-calculates the delta between your current position and the target `position_size`, then sends only the incremental order.

| Param                                           | Required | Notes                                 |
| ----------------------------------------------- | -------- | ------------------------------------- |
| `symbol`, `quantity`, `action`, `position_size` | Yes      | `position_size` = your target net qty |
| Rest                                            | No       | Same defaults as `place_order`        |

**Prompts:**

* _"Square off my TATAMOTORS intraday position to zero"_
* _"Scale my YESBANK position to 100 shares long"_

***

#### `place_basket_order`

Fire multiple orders in one call. Each basket entry carries its own `symbol`, `exchange`, `action`, `quantity`, `pricetype`, `product`.

**Prompts:**

* _"Place a basket: buy 1 BHEL and sell 1 ZOMATO, both market MIS on NSE"_
* _"Build a basket of SBIN, HDFCBANK and ICICIBANK buys, 10 shares each, CNC"_

***

#### `place_split_order`

Break a large order into equal chunks (helpful for low-liquidity names or to avoid freeze limits).

**Prompts:**

* _"Sell 500 YESBANK in slices of 50, market orders"_
* _"Split 1200 NIFTY lots across 100-lot chunks"_

***

#### `place_options_order`

Single-leg option order using offset-based strike selection (ATM / ITM1–ITM50 / OTM1–OTM50). The server resolves the strike against the live option chain.

| Param                                                                   | Required | Notes                                                            |
| ----------------------------------------------------------------------- | -------- | ---------------------------------------------------------------- |
| `underlying`, `exchange`, `offset`, `option_type`, `action`, `quantity` | Yes      | —                                                                |
| `expiry_date`                                                           | No       | Optional if underlying includes expiry (e.g., `NIFTY28OCT25FUT`) |
| `price_type`, `product`, `price`, `trigger_price`                       | No       | Same as `place_order`                                            |

> **Lot size note**: if you don't know it, just ask — the assistant will pull `lotsize` from `get_option_symbol` first, then size the quantity correctly.

**Prompts:**

* _"Buy 1 lot NIFTY ATM CE expiring 28NOV25"_
* _"Short 2 lots BANKNIFTY OTM3 PE for next weekly expiry"_

***

#### `place_options_multi_order`

Multi-leg option strategies (up to 20 legs). BUY legs are fired first for margin efficiency, then SELL legs. Supports per-leg overrides for `expiry_date`, `pricetype`, `price`, `product`, etc. — perfect for calendar / diagonal spreads.

**Prompts:**

* _"Place an iron condor on NIFTY 28NOV25 at OTM5 and OTM10 strikes, 1 lot each, NRML"_
* _"Build a long straddle on BANKNIFTY ATM for 25NOV25 expiry with limit orders at 250"_
* _"Diagonal NIFTY spread: buy ITM2 CE 30DEC25, sell OTM2 CE 25NOV25, 1 lot"_

***

#### `modify_order`

Change price / quantity / type / trigger on a working order.

| Param                                                                      | Required | Notes                                                                   |
| -------------------------------------------------------------------------- | -------- | ----------------------------------------------------------------------- |
| `order_id`, `symbol`, `action`, `exchange`, `product`, `quantity`, `price` | Yes      | `price` is mandatory per the REST spec — use current price if unchanged |
| `price_type`, `trigger_price`, `disclosed_quantity`                        | No       | Sensible defaults                                                       |

**Prompts:**

* _"Modify order 250408001002736 — change limit price to 16.5"_
* _"Increase quantity of my open NIFTY CE buy order to 2 lots"_

***

#### `cancel_order`

**Prompt:** _"Cancel order 250408001002736"_

***

#### `cancel_all_orders`

**Prompts:**

* _"Cancel every pending order I have"_
* _"Kill all open orders for strategy 'nifty scalper'"_

***

### 📊 Positions & Holdings

#### `close_all_positions`

Square off everything for a strategy.

**Prompt:** _"Close all my open positions now"_

***

#### `get_open_position`

Query the current net quantity for a specific instrument.

**Prompts:**

* _"What's my current position in NHPC NSE MIS?"_
* _"How many NIFTY futures am I long?"_

***

#### `get_position_book`

Every open position across instruments.

**Prompt:** _"Show me all open positions with unrealized P\&L"_

***

#### `get_holdings`

Delivery/CNC holdings with today's P\&L, % move, and aggregate statistics.

**Prompts:**

* _"Show my demat holdings sorted by today's % change"_
* _"What's the total unrealized P\&L on my long-term portfolio?"_

***

#### `get_funds`

Cash, collateral, realized/unrealized M2M, utilized margin.

**Prompt:** _"How much free cash do I have for trading today?"_

***

### 📋 Order Tracking

#### `get_order_status`

**Prompt:** _"Check status of order 250828000185002 — did it fill?"_

***

#### `get_order_book`

Every order today with statistics (open / complete / cancelled / rejected counts).

**Prompts:**

* _"Show today's order book"_
* _"How many of my orders got rejected today and why?"_

***

#### `get_trade_book`

Only executed fills.

**Prompt:** _"List all my executed trades today with average price"_

***

### 📈 Market Data

#### `get_quote`

LTP, bid, ask, OHLC, volume for one symbol.

**Prompts:**

* _"Get the latest quote for RELIANCE"_
* _"What's NIFTY trading at right now?"_

***

#### `get_multi_quotes`

Quotes for many symbols in one round-trip.

**Prompt:** _"Get quotes for RELIANCE, TCS, INFY, HDFCBANK and ICICIBANK"_

***

#### `get_market_depth`

Full 5-level bid/ask book plus total buy/sell qty and OI.

**Prompt:** _"Show the order book depth for SBIN"_

***

#### `get_historical_data`

OHLCV history. Two sources:

* `source="api"` (default) → live fetch from broker API
* `source="db"` → local Historify DuckDB store (1m and D stored physically; other intervals, including custom ones like 2m/3m/W/M/Q/Y, computed on-the-fly via SQL)

**Prompts:**

* _"Get 5-minute SBIN candles from 1 Apr to 8 Apr 2025"_
* _"Pull NIFTY daily data for the last 6 months from the local Historify DB"_
* _"Give me weekly aggregates of BANKNIFTY for the past year using source=db"_

***

#### `get_option_chain`

Real-time chain with CE/PE data per strike — LTP, bid/ask, OHLC, volume, OI, `lotsize`, moneyness labels. Use `strike_count=N` to limit to N strikes around ATM.

**Prompts:**

* _"Show me NIFTY option chain for 30DEC25, 10 strikes around ATM"_
* _"Full BANKNIFTY option chain for this week's expiry"_

***

### 🔍 Instrument Search & Symbols

#### `search_instruments`

Fuzzy search across exchanges by name or symbol.

**Prompts:**

* _"Search for NIFTY 26000 Dec CE"_
* _"Find all TATA stocks on NSE"_

***

#### `get_symbol_info`

Full metadata for one symbol: `brsymbol`, `lotsize`, `expiry`, `strike`, `tick_size`, `token`.

**Prompts:**

* _"Get symbol info for NIFTY30DEC25FUT on NFO"_
* _"What's the lot size for BANKNIFTY futures?"_

***

#### `get_option_symbol`

Resolve ATM/ITM/OTM offset to the exact option symbol plus `lotsize`, `tick_size`, `underlying_ltp`. Expiry optional if the underlying includes one.

**Prompts:**

* _"Get the ATM CE symbol for NIFTY expiring 28OCT25"_
* _"What's the OTM4 PE for BANKNIFTY next weekly?"_

***

#### `get_option_greeks`

Delta, Gamma, Theta, Vega, Rho + Implied Volatility using Black-76. Underlying is auto-detected — override with `underlying_symbol` / `underlying_exchange`, supply `forward_price` for custom / illiquid underlyings, and `expiry_time` for non-standard MCX contracts.

**Prompts:**

* _"Calculate greeks for NIFTY25NOV2526000CE with 6.5% interest rate"_
* _"What's the delta and IV of the ATM NIFTY CE for 28NOV25?"_

***

#### `get_synthetic_future`

Put-call parity synthetic future price — useful for illiquid futures or weekly expiries that lack a traded future.

**Prompt:** _"What's the NIFTY synthetic future price for 25NOV25?"_

***

#### `get_expiry_dates`

All tradeable expiries for an underlying.

**Prompt:** _"List all NIFTY options expiries available on NFO"_

***

#### `get_available_intervals`

Supported timeframes for `get_historical_data`.

**Prompt:** _"What intraday intervals are supported?"_

***

#### `get_instruments`

Bulk instrument master download for an exchange (or all exchanges when `exchange` is omitted). Output is paginated — default limit 500, with a `truncated` flag.

**Prompts:**

* _"Download the NFO instrument master, first 500 rows"_
* _"Get all MCX instruments available for trading"_

***

#### `get_index_symbols`

Returns the full standardized OpenAlgo index symbol list (57 NSE + 40 BSE), rolled out uniformly across every supported broker.

**Prompts:**

* _"List all NSE index symbols I can subscribe to"_
* _"Show me the BSE index list — I want to stream SENSEX50"_

***

### 💰 Margin

#### `calculate_margin`

SPAN + exposure margin for a hypothetical position set. Accepts an array of legs with `symbol`, `exchange`, `action`, `product`, `pricetype`, `quantity`.

**Prompts:**

* _"Calculate margin for 1 lot NIFTY 25000 CE buy + 1 lot 25500 CE sell, 25NOV25 expiry"_
* _"How much margin do I need for a BANKNIFTY short straddle at ATM for next week?"_

***

### 🧪 Analyzer

#### `analyzer_status`

Am I in simulated (analyzer) or live mode?

**Prompt:** _"Am I in live or analyzer mode right now?"_

***

#### `analyzer_toggle`

Flip between simulated and live trading. Analyzer mode returns `SB-xxx` pseudo-orderids without touching the broker — perfect for testing strategies end-to-end.

**Prompts:**

* _"Switch to analyzer mode before I test this strategy"_
* _"Turn off analyzer — I want to go live"_

***

### 📅 Market Calendar

#### `get_holidays`

Full holiday list for a year (year optional → defaults to current year).

**Prompts:**

* _"What are the trading holidays in 2026?"_
* _"List this year's market holidays"_

***

#### `get_timings`

Exchange open/close epoch timestamps for a given date (date optional → defaults to today).

**Prompt:** _"What are today's market timings across NSE, BFO and MCX?"_

***

#### `check_holiday`

Quick pre-trade check: is a given date a holiday on a specific exchange?

**Prompts:**

* _"Is 26 Jan 2026 a holiday on NSE?"_
* _"Is tomorrow a trading day on MCX?"_

***

### 🛠️ Utilities

#### `get_openalgo_version`

**Prompt:** _"What version of the openalgo library is running?"_

***

#### `validate_order_constants`

Quick cheat-sheet of valid exchanges, product types, price types, actions, and intervals — useful when the model wants to double-check a parameter before sending an order.

**Prompt:** _"Remind me of the valid product types and price types"_

***

#### `send_telegram_alert`

Push a Telegram notification via the OpenAlgo Telegram bot (must be configured in OpenAlgo settings first). Supports `priority` 1–10.

**Prompts:**

* _"Send me a Telegram alert: NIFTY crossed 26000, priority 8"_
* _"Ping me on Telegram if my NIFTY CE fills"_

***

### 🧠 Worked Multi-Tool Workflows

Real strength shows when the assistant chains tools on its own. Example prompts:

**1. End-to-end iron condor (analyzer mode):**

> _"Set up a NIFTY iron condor for next week's expiry. Find the expiry, pull the option chain, use OTM5 strikes on both sides, calculate the margin required, and — only if margin is under ₹1L — place it in analyzer mode using 1 lot per leg."_

The assistant will chain: `get_expiry_dates` → `get_option_chain` → `get_option_symbol` (for lot size) → `calculate_margin` → `analyzer_status` / `analyzer_toggle` → `place_options_multi_order`.

**2. Pre-market checklist:**

> _"Before I start trading: is the market open today on NSE and MCX, what's my free cash, what's my current position book, and what's NIFTY spot right now?"_

Chains: `check_holiday` → `get_timings` → `get_funds` → `get_position_book` → `get_quote`.

**3. Options greeks scan:**

> _"Pull the NIFTY option chain for 25NOV25 within 5 strikes of ATM, then compute greeks for the ATM CE and PE with 6.5% interest rate — tell me which has higher vega."_

Chains: `get_option_chain` → `get_option_symbol` (ATM) × 2 → `get_option_greeks` × 2.

**4. Square-off with Telegram confirmation:**

> _"Square off everything, cancel all pending orders, then send me a Telegram alert summarizing what got closed with the realized P\&L."_

Chains: `cancel_all_orders` → `close_all_positions` → `get_trade_book` → `send_telegram_alert`.

***

### Quick Prompt Patterns

| Intent        | Prompt pattern                                                 |
| ------------- | -------------------------------------------------------------- |
| Status check  | _"What's my {thing}?"_                                         |
| Single action | _"{Buy/Sell} {qty} {symbol} at {price}"_                       |
| Multi-leg     | _"Build a {strategy} on {underlying} {expiry} with {offsets}"_ |
| Safety-first  | _"In analyzer mode, {do the thing}"_                           |
| Conditional   | _"Only if {condition}, then {action}"_                         |
| Research      | _"Show me {chain/greeks/history} and recommend {levels}"_      |

***

### Safety Tips

* Start in **analyzer mode** (`analyzer_toggle True`) while you get comfortable — orders look real but never leave OpenAlgo.
* Use phrases like _"only if margin is under X"_ or _"ask me to confirm before placing"_ — the assistant will pause for your OK.
* Use a unique `strategy` name per use-case (e.g., _"use strategy 'nifty scalper'"_) so MCP-driven activity is cleanly separable from manual orders in logs.
* For live trading, set up the OpenAlgo Telegram bot and ask the assistant to _"send a Telegram alert after every order fill"_ — you get a realtime feed without staring at the screen.

***

### Related

* MCP Server Setup Guide — install, configure Claude / Cursor / Windsurf, broker prerequisites
* OpenAlgo Symbol Format — how equity / future / option symbols are constructed
* API Documentation — underlying REST endpoints each MCP tool wraps
