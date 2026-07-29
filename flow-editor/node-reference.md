# Node Reference

Every node type Flow provides — 60 in total — with its configuration fields,
what it writes to its output variable, and the traps specific to it.

Fields marked **required** must be set or the node fails at run time. Every
data node takes an `outputVariable`; if you leave it blank the node still runs
but nothing downstream can read it.

Text fields accept `{{variable}}` interpolation. **Trigger fields do not** —
a trigger is the entry point, so nothing has been computed yet.

---

## Triggers

A workflow must have exactly **one** trigger. If you add a second, the
executor takes the first one it finds and everything downstream of the other
never runs, silently. See [Concepts](concepts.md#triggers).

### `start` — Schedule

| Field | Values | Notes |
| --- | --- | --- |
| `scheduleType` | `once`, `daily`, `weekly`, `interval` | Required. |
| `time` | `"HH:MM"` | For `once` / `daily` / `weekly`. IST. |
| `days` | `[1,2,3,4,5]` | For `weekly`. 1 = Monday. |
| `executeAt` | ISO datetime | For `once`. |
| `intervalValue` + `intervalUnit` | number + `seconds`/`minutes`/`hours` | For `interval`. |
| `intervalMinutes` | number | Older form of the above; still honoured. |
| `marketHoursOnly` | boolean | Skips runs outside market hours. |

The scheduler is IST-based and survives restarts (jobs persist in
`flow_apscheduler_jobs`). Overlapping runs of the same workflow are suppressed
(`max_instances: 1`), so a slow run delays the next tick rather than stacking.

### `priceAlert` — Price Alert

| Field | Values | Notes |
| --- | --- | --- |
| `symbol`, `exchange` | | Required. |
| `condition` | `above`, `below`, `crosses_above`, `crosses_below` | |
| `price` | number | The level to watch. |
| `enabled` | boolean | |

`crosses_above` / `crosses_below` are edge-triggered and need a previous tick,
so the first evaluation after activation cannot fire them.

### `webhookTrigger` — Webhook

Fires when its URL is POSTed. The payload is readable as
`{{webhook.<field>}}` — for example `{{webhook.symbol}}`,
`{{webhook.action}}`. This is how TradingView, ChartInk, Amibroker and Excel
drive a Flow workflow.

The node has no configuration beyond the generated `webhookId` / `webhookUrl`,
which the editor fills in. Anything in the JSON body is available; nothing is
validated for you, so guard on the fields you rely on.

### `orderUpdateTrigger` — Order Update

Fires when an order's status changes, pushed from the same event stream as the
account WebSocket feed — no polling.

| Field | Values | Notes |
| --- | --- | --- |
| `orderId` | string | Watch one order. Empty = any. |
| `symbol`, `exchange` | | Optional filters. |
| `status` | `any`, `open`, `trigger pending`, `complete`, `rejected`, `cancelled` | |
| `trigger` | `once`, `every_time` | `once` removes the watch after it fires. |

The payload is exposed as `{{webhook.orderid}}`, `{{webhook.order_status}}`,
`{{webhook.filled_quantity}}`, `{{webhook.average_price}}`,
`{{webhook.rejection_reason}}`.

### `httpRequest` — HTTP Request

Can act as a trigger or as an action mid-graph. Calls an external URL and
stores the response.

---

## Order placement

Every order node calls the same service layer as `/api/v1/`, so Analyzer mode,
Action Center approval, rate limiting, and Telegram/WhatsApp alerts behave
exactly as they do for an API order.

### `placeOrder` — Place Order

| Field | Values | Notes |
| --- | --- | --- |
| `symbol`, `exchange` | | Required. |
| `action` | `BUY`, `SELL` | Required. |
| `quantity` | number | **Lots** for F&O, shares for equity. |
| `priceType` | `MARKET`, `LIMIT`, `SL`, `SL-M` | Default `MARKET`. |
| `product` | `CNC`, `NRML`, `MIS` | |
| `price`, `triggerPrice`, `disclosedQuantity` | number | `price` required for `LIMIT`; `triggerPrice` for `SL`/`SL-M`. |

Output: `{status, orderid}`

### `smartOrder` — Smart Order

Same fields as `placeOrder` plus `positionSize`: the **target** net position.
The node computes the delta and places only the difference. `positionSize: 0`
flattens. This is the safe way to "get to" a position without tracking what
you already hold.

Output: `{status, orderid}`

### `optionsOrder` — Options Order

Resolves the contract for you, then places it.

| Field | Values | Notes |
| --- | --- | --- |
| `underlying` | e.g. `NIFTY` | Required. |
| `exchange` | `NFO`, `BFO` | The F&O exchange, not `NSE_INDEX`. |
| `expiryDate` | `"04-AUG-26"` or blank | Blank = current weekly. |
| `offset` | integer | Strikes from ATM. `0` = ATM, `+2` = two strikes OTM for a call. |
| `optionType` | `CE`, `PE` | |
| `action`, `quantity`, `priceType`, `product`, `price` | | As `placeOrder`. |
| `splitSize` | number | Splits into child orders of this size for freeze limits. |

Output: `{status, orderid, symbol, exchange, underlying, underlying_ltp, offset, option_type, mode}`

### `optionsMultiOrder` — Options Multi-Order

Places several option legs in one call — spreads, straddles, iron condors.

| Field | Notes |
| --- | --- |
| `strategy` | The **structure** name (`custom`, `straddle`, …). This is *not* the strategy tag. |
| `underlying`, `exchange`, `expiryDate` | Shared by all legs. |
| `legs` | Array of `{offset, optionType, action, quantity, priceType, product}`. |

Output: `{status, results: [{...}]}`

Legs are placed together and reported in one completion event.

### `basketOrder` — Basket Order

Several unrelated symbols in one submission.

| Field | Notes |
| --- | --- |
| `orders` | Array of `{symbol, exchange, action, quantity, priceType, product, price}`. |

Output: `{status, results: [{symbol, exchange, product, status, orderid}]}`

### `splitOrder` — Split Order

One large order broken into child orders of `splitSize`, `delayMs` apart, to
stay under exchange freeze quantity.

Output: `{status, results: [{order_num, quantity, status, orderid}]}`

### `modifyOrder` — Modify Order

`orderId` plus `newQuantity`, `newPrice`, `newTriggerPrice`. Pair it with
`orderUpdateTrigger` to react to a partial fill.

### `cancelOrder` / `cancelAllOrders` / `closePositions`

`cancelOrder` takes an `orderId`. `cancelAllOrders` takes nothing and cancels
every open order. `closePositions` flattens everything, optionally narrowed by
`exchange` and `product`.

These are account-wide, **not** strategy-scoped. `cancelAllOrders` in one
workflow cancels orders another workflow placed.

---

## Conditions and logic

A condition node emits a boolean consumed by edge routing — it has **no**
output variable. Reference its *inputs* downstream, not its result.

Each has a `true` and a `false` source handle. A condition that **errors**
takes neither branch, so a typo in a variable path cannot fire the else-path
by accident.

### `priceCondition` — Price Condition

| Field | Values |
| --- | --- |
| `symbol`, `exchange` | |
| `field` | `ltp`, `open`, `high`, `low`, `close`, `prev_close`, `volume` |
| `operator` | `>`, `<`, `>=`, `<=`, `==`, `!=` |
| `value` | number, or `{{variable}}` |

### `varCondition` — Variable Condition

Compares any two values, each a literal or `{{path}}`. This is the general
comparison node — use it whenever `priceCondition` is too narrow.

| Field | Notes |
| --- | --- |
| `leftValue`, `rightValue` | Literal or `{{path}}`. |
| `operator` | `>`, `<`, `>=`, `<=`, `==`, `!=` |

If a path does not resolve, the condition errors and takes neither branch —
which is what you want, but it means a silently missing value looks like
"nothing happened". Check the run log.

### `timeWindow` — Time Window

`startTime`, `endTime` (`"HH:MM"`, IST), optional `days`, and
`invertCondition` to mean "outside this window".

### `timeCondition` — Time Condition

`conditionType`, `targetTime`, `operator` — for "is it past 14:00" style
checks against a single time rather than a range.

### `positionCheck` — Position Check

| Field | Values |
| --- | --- |
| `symbol`, `exchange`, `product` | |
| `condition` | `has_position`, `no_position`, `quantity_above`, `quantity_below`, `pnl_above`, `pnl_below` |
| `threshold` | number |

This asks the **broker**, so it is the correct way to answer "am I already in
this trade" — Flow keeps no memory between runs.

### `fundCheck` — Fund Check

`minAvailable` — passes when available cash is at least this.

### `andGate` / `orGate` / `notGate`

Combine conditions. A gate waits until **every** connected input has been
evaluated before it fires, and fires exactly once per run.

Connect condition outputs into the gate, and the gate's output onward. Gates
have no configuration.

---

## Market data

See [Market Data and Timeframes](market-data.md) for the bar limits and
caching behaviour that apply across these nodes.

### `getQuote` — Get Quote

`symbol`, `exchange` → `{status, data: {ltp, open, high, low, prev_close, volume, oi, bid, ask}}`

`data.high` and `data.low` are **today's session extremes**, which is what
makes stateless "has price already touched this level today" tests possible.

### `multiQuotes` — Multi Quotes

Several symbols in one call → `{status, results: [{symbol, exchange, data: {...}}]}`

Prefer this over several `getQuote` nodes: quotes are **not** cached and some
brokers allow only ~1 quote request per second.

### `getDepth` — Get Depth

→ `{status, data: {bids: [{price, quantity}], asks: [...], ltp, totalbuyqty, totalsellqty}}`

### `history` — Historical Data

| Field | Notes |
| --- | --- |
| `symbol`, `exchange` | |
| `interval` | `1m`, `3m`, `5m`, `15m`, `30m`, `1h`, `D`, `W`, `M` — broker-dependent, check with `intervals`. |
| `days` | Calendar days back. Capped so the result never exceeds **200 bars**. |

→ `{status, data: [{timestamp, open, high, low, close, volume, oi}]}`

Timestamps are **epoch seconds**, not ISO strings.

### `priorPeriodOhlc` — Prior Period OHLC

The completed previous day/week/month — the reliable way to get PDH/PDL
without off-by-one errors from today's partial candle.

| Field | Values |
| --- | --- |
| `symbol`, `exchange` | |
| `period` | `day`, `week`, `month` |
| `source` | `broker` or `db` (Historify) |

→ `{status, symbol, exchange, period, date, open, high, low, close, volume, pdh, pdl, pdc}`

### `barOffset` — Bar Offset

One specific bar counted back from the latest — "the close 20 hours ago", "the
high 3 bars ago".

`offsetBars: 0` is the most recent bar; `1` is the one before it.

→ `{status, symbol, exchange, offsetBars, timestamp, open, high, low, close, volume}`

### `indicator` — Technical Indicator

All 118 single-symbol indicators. Fully covered in
[Indicators](indicators.md).

| Field | Notes |
| --- | --- |
| `indicatorName` | e.g. `ema`, `rsi`, `macd`, `supertrend`. |
| `params` | Object of the indicator's own parameters, e.g. `{"period": 20}`. |
| `interval` | Any interval the broker supports. |
| `lookbackBars` / `tailBars` | How much history to compute over. |
| `offsetBars` | Which past value `at_offset` refers to. |
| `sourceField` | `close` (default), `open`, `high`, `low`, `volume`. |
| `sourceSeries` | Feed another indicator's output in — this is how nesting works. |

→ `{status, indicator, nested, inputs, params, outputs, latest, previous, at_offset, series, offset_bars, bars_used}`

Single-output indicators expose `value`; multi-output expose `out0`, `out1`, …

### `strategyPnl` — Strategy P&L

Realized, unrealized and total P&L for **one strategy**, not the whole
account — so a workflow can exit on its own performance even when other
strategies hold the same contract.

| Field | Notes |
| --- | --- |
| `strategy` | Defaults to this workflow's name, which is also the tag its order nodes apply. Usually leave blank. |

→ `{status, strategy, realized, today_realized, unrealized, total, open_quantity, unpriced_legs, legs: [...]}`

`unpriced_legs` counts open legs with no live price; those are **excluded**
from `unrealized`. A non-zero value means the total is incomplete — guard on
it before acting. If the position book itself is unavailable the node returns
an error rather than a misleading zero.

### `openPosition` — Open Position

`symbol`, `exchange`, `product` → `{status, quantity}`. Signed: negative is
short.

### `intervals` — Supported Intervals

→ `{status, data: {seconds, minutes, hours, days, weeks, months}}`

Use it to confirm your broker supports an interval before relying on it. Not
every broker supports every timeframe.

---

## Symbols, expiries and option chains

### `symbol` — Symbol Info

→ `{status, data: {symbol, brsymbol, lotsize, tick_size, expiry, strike, token, ...}}`

`data.lotsize` is how you size an F&O order correctly.

### `expiry` — Expiry Dates

`symbol`, `exchange`, `instrumenttype` (`options` / `futures`)
→ `{status, message, data: ["04-AUG-26", ...]}`, nearest first.

### `optionSymbol` — Option Symbol

Resolves an ATM-relative strike to a tradable symbol without placing anything.

`underlying`, `exchange`, `expiryDate`, `offset`, `optionType`
→ `{status, symbol, exchange, lotsize, tick_size, freeze_qty, underlying_ltp}`

Note this shape is **flat** — `{{os.symbol}}`, not `{{os.data.symbol}}`.

### `optionChain` — Option Chain

`underlying`, `exchange`, `expiryDate`, `strikeCount`
→ `{status, underlying, underlying_ltp, expiry_date, atm_strike, chain: [{strike, ce: {...}, pe: {...}}]}`

### `syntheticFuture` — Synthetic Future

→ `{status, underlying, expiry, atm_strike, synthetic_future_price, underlying_ltp}`

---

## Account and orders

### `funds` — Funds

→ `{status, data: {availablecash, collateral, m2mrealized, m2munrealized, utiliseddebits}}`

### `orderBook` — Order Book

→ `{status, data: {orders: [...], statistics: {...}}}`

`data.statistics.total_buy_orders` is the usual stateless "have I already
entered today" guard — the order book resets daily and keeps the record after
a position closes.

### `tradeBook` — Trade Book

→ `{status, data: [{tradeid, orderid, symbol, average_price, ...}]}`

### `positionBook` — Position Book

→ `{status, data: [{symbol, quantity, average_price, ltp, pnl, ...}], total_pnl}`

Account-wide and netted per `(symbol, exchange, product)` — it cannot tell you
which strategy opened a position. Use `strategyPnl` for that.

### `holdings` — Holdings

→ `{status, data: {holdings: [...], statistics: {...}}}`

Note the nesting: `{{hd.data.holdings[0].symbol}}`, not `{{hd.data[0].symbol}}`.

### `margin` — Margin Calculator

Pre-trade margin for a proposed order or basket.

| Field | Notes |
| --- | --- |
| `positions` | Array for a multi-leg estimate. |
| `symbol`, `exchange`, `action`, `quantity`, `product`, `priceType` | For a single order. |

### `getOrderStatus` — Order Status

| Field | Notes |
| --- | --- |
| `orderId` | Required. |
| `waitForCompletion` | Blocks the run until the order reaches a terminal state. |

→ `{status, data: {order_status, average_price, quantity, ...}}`

`waitForCompletion` blocks the whole run. For anything but a fast market
order, prefer `orderUpdateTrigger` in a separate workflow.

### `holidays` / `timings`

`holidays` takes a `year` → the exchange holiday list. `timings` takes a
`date` → market open/close for that day. Use them to avoid firing on a
holiday.

---

## Streaming

These maintain a WebSocket subscription and pass the latest tick to their
output variable. If WebSocket is unavailable they fall back to a single REST
call — identical from the workflow's point of view.

| Node | Output |
| --- | --- |
| `subscribeLtp` | The float LTP directly (not wrapped in an object). |
| `subscribeQuote` | `{ltp, open, high, low, close, volume, ...}` |
| `subscribeDepth` | `{bids: [...], asks: [...], totalbuyqty, totalsellqty, ltp}` |
| `unsubscribe` | `streamType`: `ltp` / `quote` / `depth` / `all`. Empty `symbol` = all symbols. |

---

## Utilities

### `variable` — Variable

| Field | Values |
| --- | --- |
| `variableName` | |
| `operation` | `set`, `increment`, `decrement`, `extract` |
| `value` | Literal or `{{path}}`. |
| `sourceVariable` + `jsonPath` | For `extract`. |

**Variables do not survive the run.** An `increment` counts within one
execution only; it is reset on the next tick. Anything that must persist has
to come from the broker.

### `mathExpression` — Math Expression

`expression` — arithmetic over interpolated values, e.g.
`"{{q.data.ltp}} * 1.02"`. Result goes to `outputVariable`.

Arithmetic and comparison only: no string manipulation, no date arithmetic, no
function calls.

### `log` — Log

`message` (interpolated) and `level` (`info`, `warning`, `error`). Written to
the workflow's execution log — the primary way to debug a run.

### `telegramAlert` — Telegram Alert

`message` (interpolated) and `username`. Requires the Telegram bot to be
configured and that username linked.

### `whatsappAlert` — WhatsApp Alert

`to` and `message`. Requires the WhatsApp bot to be paired.

### `delay` — Delay

`delayValue` + `delayUnit` (`seconds` / `minutes` / `hours`), or `delayMs`.

**Blocking, and uncapped.** The run sleeps and holds its execution slot the
whole time. Do not use it to wait out a trading session — use a second
workflow on its own schedule.

### `waitUntil` — Wait Until

`targetTime` (`"HH:MM"`). Returns immediately with `{status, waited: false}`
if the time has already passed today.

Same warning, more so: `waitUntil` set to `15:20` from a 09:20 run blocks that
run for six hours. Prefer a scheduled trigger.

### `group` — Group

Visual grouping only. No execution behaviour.

---

## Node count by category

| Category | Count | Nodes |
| --- | --- | --- |
| Triggers | 5 | `start`, `priceAlert`, `webhookTrigger`, `orderUpdateTrigger`, `httpRequest` |
| Order placement | 10 | `placeOrder`, `smartOrder`, `optionsOrder`, `optionsMultiOrder`, `basketOrder`, `splitOrder`, `modifyOrder`, `cancelOrder`, `cancelAllOrders`, `closePositions` |
| Conditions and logic | 9 | `priceCondition`, `varCondition`, `timeWindow`, `timeCondition`, `positionCheck`, `fundCheck`, `andGate`, `orGate`, `notGate` |
| Market data | 10 | `getQuote`, `multiQuotes`, `getDepth`, `history`, `priorPeriodOhlc`, `barOffset`, `indicator`, `strategyPnl`, `openPosition`, `intervals` |
| Symbols and options | 5 | `symbol`, `expiry`, `optionSymbol`, `optionChain`, `syntheticFuture` |
| Account and orders | 9 | `funds`, `orderBook`, `tradeBook`, `positionBook`, `holdings`, `margin`, `getOrderStatus`, `holidays`, `timings` |
| Streaming | 4 | `subscribeLtp`, `subscribeQuote`, `subscribeDepth`, `unsubscribe` |
| Utilities | 8 | `variable`, `mathExpression`, `log`, `telegramAlert`, `whatsappAlert`, `delay`, `waitUntil`, `group` |
| **Total** | **60** | |

For the exact JSON schema of every field — the format an AI needs to generate
an importable workflow — see
[`docs/prompt/flow-import-format.md`](https://github.com/marketcalls/openalgo/blob/main/docs/prompt/flow-import-format.md)
in the main repository.
