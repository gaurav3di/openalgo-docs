# Node Reference

Every node type Flow provides, 61 in total, with its configuration fields,
what it writes to its output variable, and the traps specific to it.

Fields marked **required** must be set or the node fails at run time. Every
data node takes an `outputVariable`; if you leave it blank the node still runs
but nothing downstream can read it.

Text fields accept `{{variable}}` interpolation. **Trigger fields do not**:
a trigger is the entry point, so nothing has been computed yet.

---

## Triggers

A workflow must have exactly **one** trigger, and there are exactly four:
`start`, `priceAlert`, `webhookTrigger`, and `orderUpdateTrigger`. A second
trigger is rejected by strict validation with a `multiple_triggers` error, so
**Run Now**, activation, and import all refuse the graph. See
[Concepts](concepts.md#triggers).

### `start`: Schedule

| Field | Values | Notes |
| --- | --- | --- |
| `scheduleType` | `once`, `daily`, `weekly`, `interval`, `manual` | Required. `manual` registers no job at all. |
| `time` | `"HH:MM"` | For `daily` and `weekly` only. IST. |
| `days` | `[0,1,2,3,4]` | For `weekly`. **0 = Monday**, 6 = Sunday. |
| `executeAt` | date | For `once`. The editor writes a bare `YYYY-MM-DD`, so a `once` schedule fires at midnight and the panel's Time box does not apply to it. |
| `intervalValue` + `intervalUnit` | number + `seconds`/`minutes`/`hours` | For `interval`. |
| `marketHoursOnly` | boolean | Gates runs to the exchange trading session. |
| `marketHoursExchange` | exchange code | Which calendar to consult. Default NSE. |
| `marketHoursStart` / `marketHoursEnd` | `"HH:MM"` | IST overrides that narrow or extend the session clock. They never reopen a holiday. |

`intervalMinutes` appears in older exports and is read by nothing. A workflow
relying on it schedules at the default of one minute.

Jobs persist in the `flow_apscheduler_jobs` table and survive restarts. The
scheduler uses the server process local timezone for cron and date triggers;
only the market-hours gate is explicitly IST, so run OpenAlgo with the host
clock set to IST.

Overlapping runs are **dropped, not deferred**. The job defaults are
`coalesce: true`, `max_instances: 1`, and a 60-second misfire grace, and
`execute_workflow` takes the per-workflow lock without blocking and returns
`already_running`. A tick that arrives during a slow run is lost.

### `priceAlert`: Price Alert

| Field | Values | Notes |
| --- | --- | --- |
| `symbol`, `exchange` | | Required. |
| `condition` | see below | 13 canonical values. |
| `price` | number | The level to watch. Required for the level and crossing conditions. |
| `priceLower` + `priceUpper` | number | Required for the four channel conditions. `priceLower` must be below `priceUpper`. |
| `percentage` | number | Required for `moving_up_percent` / `moving_down_percent`. |
| `trigger` | `once`, `every_time` | Default `once`. |
| `expiration` | `none`, `1h`, `4h`, `1d`, `1w` | Retires the alert after this window. |

The 13 conditions the monitor evaluates:

| Group | Values | Needs |
| --- | --- | --- |
| Level | `greater_than`, `less_than` | `price` |
| Crossing | `crossing`, `crossing_up`, `crossing_down` | `price` |
| Channel | `entering_channel`, `inside_channel`, `exiting_channel`, `outside_channel` | `priceLower`, `priceUpper` |
| Movement | `moving_up`, `moving_down` | nothing extra |
| Percentage move | `moving_up_percent`, `moving_down_percent` | `percentage` |

`above`, `below`, `crosses_above`, and `crosses_below` are accepted as aliases
of `greater_than`, `less_than`, `crossing_up`, and `crossing_down`.

**A crossing condition can fire on its very first evaluation.** With no
previous tick recorded, `crossing_up` falls back to a plain level test
(`current > target`) rather than returning false. An alert armed while price
is already above its level therefore fires on the next poll, not on the next
genuine crossing.

A `once` alert does not merely remove its own watch when it fires: it
**deactivates the whole workflow**. Reactivate it from the editor to arm it
again.

The `enabled` field exists in older exports and is read by nothing.

### `webhookTrigger`: Webhook

Fires when its URL is POSTed. The payload is readable as
`{{webhook.<field>}}`, for example `{{webhook.symbol}}`,
`{{webhook.action}}`. This is how TradingView, ChartInk, Amibroker and Excel
drive a Flow workflow.

The URL is not stored on the node. It is derived from the workflow's own
`webhook_token`, which the editor fetches from
`GET /api/workflows/<id>/webhook`, so the node itself has no `webhookId` or
`webhookUrl` field. Its declared fields are `label`, `symbol`, and `exchange`,
none of which the executor reads. Anything in the JSON body is available;
nothing is validated for you, so guard on the fields you rely on.

### `orderUpdateTrigger`: Order Update

Fires when an order's status changes, pushed from the same event stream as the
account WebSocket feed, no polling.

| Field | Values | Notes |
| --- | --- | --- |
| `orderId` | string | Watch one order. Must be a literal: `{{...}}` is rejected outright. |
| `symbol`, `exchange` | | Filters. |
| `status` | `any`, `open`, `trigger pending`, `complete`, `rejected`, `cancelled` | Default `complete`. |
| `trigger` | `once`, `every_time` | Default `once`. |

**At least one of `orderId` or `symbol` is mandatory.** Activating with both
blank fails with "orderUpdateTrigger needs an Order ID or a Symbol to watch";
there is no "watch every order" mode.

A `once` trigger deactivates the whole workflow when it fires, not just the
watch.

The payload is exposed as `{{webhook.orderid}}`, `{{webhook.order_status}}`,
`{{webhook.filled_quantity}}`, `{{webhook.average_price}}`,
`{{webhook.rejection_reason}}`.

### `httpRequest`: HTTP Request

`httpRequest` is **not a trigger**, despite sitting beside them in the node
list. It is a utility node that calls an external URL mid-graph and stores the
response. A workflow whose only entry point is an `httpRequest` fails
validation with `no_trigger`.

Its `timeout` field is in **milliseconds**: the default is 30000 and values
are capped at 60000.

---

## Order placement

Every order node calls the same service layer as `/api/v1/`, so Analyzer mode,
Action Center approval, rate limiting, and Telegram/WhatsApp alerts behave
exactly as they do for an API order.

### `placeOrder`: Place Order

| Field | Values | Notes |
| --- | --- | --- |
| `symbol`, `exchange` | | Required. |
| `action` | `BUY`, `SELL` | Required. |
| `quantity` | number | **Contract units, never lots.** For an NFO order this is lots multiplied by lot size, which you compute yourself. |
| `priceType` | `MARKET`, `LIMIT`, `SL`, `SL-M` | Default `MARKET`. |
| `product` | `CNC`, `NRML`, `MIS` | Default `MIS`. |
| `price`, `triggerPrice` | number | `price` required for `LIMIT`; `triggerPrice` for `SL`/`SL-M`. |

Output: `{status, orderid}`

`optionsOrder` and `optionsMultiOrder` are the only nodes that take lots and
multiply by the contract lot size for you. `placeOrder`, `smartOrder`,
`splitOrder`, and `basketOrder` all pass `quantity` through untouched.

`disclosedQuantity` appears in the node type definition but the executor never
sends it, so it is always 0.

### `smartOrder`: Smart Order

Takes `symbol`, `exchange`, `action`, `quantity`, `product`, `priceType`, and
`positionSize`: the **target** net position. The node computes the delta and
places only the difference. `positionSize: 0` flattens. This is the safe way
to reach a position without tracking what you already hold.

**The executor does not forward `price` or `triggerPrice` on this node.** A
`LIMIT` or `SL` smart order therefore reaches the broker with `price: 0`, and
no guard catches it. Use `smartOrder` for MARKET sizing and `placeOrder` when
you need a price.

Output: `{status, orderid}`

### `optionsOrder`: Options Order

Resolves the contract for you, then places it.

| Field | Values | Notes |
| --- | --- | --- |
| `underlying` | e.g. `NIFTY` | Required. |
| `expiryType` | `current_week`, `next_week`, `current_month`, `next_month` | Default `current_week`. Any other value errors. |
| `offset` | `ATM`, `ITM1`-`ITM50`, `OTM1`-`OTM50` | A **string**, not a number. `0` and `2` resolve nothing. |
| `optionType` | `CE`, `PE` | |
| `action` | `BUY`, `SELL` | |
| `quantity` | number | **Lots.** The node reads the contract lot size and multiplies for you. |
| `priceType`, `product`, `price`, `triggerPrice` | | `product` defaults to `NRML`. |
| `splitSize` | number | Splits into child orders of this size for freeze limits. |

There is no usable `exchange` field and no `expiryDate` field. The node
derives both exchanges from the underlying: `SENSEX`, `BANKEX`, and
`SENSEX50` resolve to `BSE_INDEX` and `BFO`, everything else to `NSE_INDEX`
and `NFO`. A workflow that sets `"expiryDate": "04-AUG-26"` silently gets the
current-week expiry.

Output when `splitSize` is 0 (the default):
`{status, orderid, symbol, exchange, underlying, underlying_ltp, offset, option_type, mode}`

Output when `splitSize` is greater than 0:
`{status, symbol, exchange, underlying, underlying_ltp, offset, option_type, total_quantity, split_size, results: [...]}`

**The split shape has no `orderid`**, so `{{o.orderid}}` is undefined exactly
when you use the split feature. Read `{{o.results}}` instead.

### `optionsMultiOrder`: Options Multi-Order

Places several option legs in one call, spreads, straddles, iron condors.

| Field | Notes |
| --- | --- |
| `strategy` | The **structure** name, one of `straddle`, `strangle`, `iron_condor`, `bull_call_spread`, `bear_put_spread`, `custom`. This is *not* the strategy tag. |
| `underlying` | Shared by all legs. The exchange is derived from it, as in `optionsOrder`. |
| `expiryType` | `current_week`, `next_week`, `current_month`, `next_month`. There is no `expiryDate` field. |
| `action` | Default `SELL`. Applied to generated legs. |
| `quantity` | Lots per leg. |
| `product`, `priceType`, `price` | Node-level defaults for generated legs. |
| `strangleWidth` | Default `OTM2`. Used by the `strangle` structure. |
| `legs` | For `custom`: array of `{offset, optionType, action, quantity, priceType, product, price, triggerPrice, splitSize}`. |

BUY legs are placed before SELL legs for margin efficiency. A generated
(non-`custom`) structure **rejects `SL` and `SL-M`** outright.

Output: `{status, underlying, underlying_ltp, results: [{...}]}`

### `basketOrder`: Basket Order

Several unrelated symbols in one submission.

| Field | Notes |
| --- | --- |
| `orders` | A newline-delimited **string**, one order per line, `SYMBOL,EXCHANGE,ACTION,QTY`. |
| `priceType`, `product` | Node-level, applied to every row. They are not per order. |
| `basketName` | Default `flow_basket`. This becomes the strategy tag. |

A per-row `price` is not parsed: every row uses the node-level `priceType`. An
array of objects is accepted as a legacy fallback but bypasses interpolation
and validation, so prefer the string form.

Output: `{status, results: [{symbol, exchange, product, status, orderid}]}`

### `splitOrder`: Split Order

One large order broken into child orders of `splitSize`, to stay under the
exchange freeze quantity. Fields: `symbol`, `exchange`, `action`, `quantity`,
`splitSize`, `priceType`, `product`.

Spacing between child orders comes from the global `ORDER_RATE_LIMIT` setting
(100 ms by default), not from the node. `delayMs` appears in older exports and
is read by nothing.

Output: `{status, results: [{order_num, quantity, status, orderid}]}`

### `modifyOrder`: Modify Order

`orderId` plus `newQuantity`, `newPrice`, `newTriggerPrice`. Pair it with
`orderUpdateTrigger` to react to a partial fill.

### `cancelOrder` / `cancelAllOrders` / `closePositions`

`cancelOrder` takes an `orderId`. It is the one order node that does **not**
store its result, so an `outputVariable` on it stays undefined.

`cancelAllOrders` takes nothing and cancels every open order.

`closePositions` behaves in two distinct ways:

* With `symbol` set, it closes that one position, honouring `exchange` and
  `product` and carrying the strategy tag.
* With `symbol` blank, it closes **everything**, in every exchange and
  product, including overnight NRML and CNC holdings. `exchange` and `product`
  filter nothing on their own.

`cancelAllOrders` and a symbol-less `closePositions` are account-wide, **not**
strategy-scoped: run in one workflow they affect orders and positions another
workflow placed.

---

## Conditions and logic

A condition node emits a boolean consumed by edge routing, it has **no**
output variable. Reference its *inputs* downstream, not its result.

Each has a `true` and a `false` source handle. A condition that **errors**
takes neither branch, so a typo in a variable path cannot fire the else-path
by accident.

### `priceCondition`: Price Condition

| Field | Values |
| --- | --- |
| `symbol`, `exchange` | |
| `field` | `ltp`, `open`, `high`, `low`, `prev_close`, `change_percent` |
| `operator` | `>`, `<`, `>=`, `<=`, `==`, `!=` |
| `value` | number, or `{{variable}}` |

Those six are the whole vocabulary. `close` and `volume` are **not** valid
here and make the node error on every run, taking neither branch. For a
close-based comparison, read it from a `history` or `barOffset` node into a
`varCondition`.

### `varCondition`: Variable Condition

Compares any two values, each a literal or `{{path}}`. This is the general
comparison node, use it whenever `priceCondition` is too narrow.

| Field | Notes |
| --- | --- |
| `leftValue`, `rightValue` | Literal or `{{path}}`. |
| `operator` | `>`, `<`, `>=`, `<=`, `==`, `!=` |

If a path does not resolve, the condition errors and takes neither branch,
which is what you want, but it means a silently missing value looks like
"nothing happened". Check the run log.

### `timeWindow`: Time Window

`startTime`, `endTime` (`"HH:MM"`, IST), and `invertCondition` to mean
"outside this window".

`days` appears in older exports and is read by nothing, so a "weekdays only"
window built from it runs every day. Gate the weekday with `{{weekday_num}}`
in a `varCondition`, or with `marketHoursOnly` on the `start` trigger.

### `timeCondition`: Time Condition

For "is it past 14:00" style checks against a single time rather than a range.

| Field | Values |
| --- | --- |
| `conditionType` | `entry`, `exit`, `custom`. Purely a label: it never changes the comparison, only the log line. |
| `targetTime` | `"HH:MM"`, IST |
| `operator` | `>=`, `<=`, `>`, `<`, `==`. `==` compares at minute precision. |

An unrecognised operator errors and takes neither branch. This node uses
`yes`/`no` source handles rather than `true`/`false`.

### `positionCheck`: Position Check

| Field | Values |
| --- | --- |
| `symbol`, `exchange`, `product` | |
| `condition` | `exists`, `not_exists`, `quantity_above`, `quantity_below`, `pnl_above`, `pnl_below` |
| `threshold` | number, default 0 |

`exists` and `not_exists` are the stored values; the editor labels them "Has
Position" and "No Position". Writing `has_position` or `no_position` into the
JSON makes the node error every run.

The node also **errors when `symbol` is blank** rather than evaluating, so it
takes neither branch.

This asks the **broker**, so it is the correct way to answer "am I already in
this trade": Flow keeps no memory between runs.

### `fundCheck`: Fund Check

`minAvailable`, passes when available cash is at least this.

### `andGate` / `orGate` / `notGate`

Combine conditions. A gate waits until **every** connected input has been
evaluated before it fires, and fires exactly once per run.

Connect condition outputs into the gate, and the gate's output onward.

`andGate` and `orGate` take an `inputCount` (2 to 5, default 2). An edge whose
`targetHandle` is `input-N` for N at or beyond the count is a hard validation
error, so raise `inputCount` before wiring a third input. Only `notGate` is
configuration-free.

---

## Market data

See [Market Data and Timeframes](market-data.md) for the bar limits and
caching behaviour that apply across these nodes.

### `getQuote`: Get Quote

`symbol`, `exchange` → `{status, data: {ltp, open, high, low, prev_close, volume, oi, bid, ask}}`

`data.high` and `data.low` are **today's session extremes**, which is what
makes stateless "has price already touched this level today" tests possible.

### `multiQuotes`: Multi Quotes

Takes a single comma-separated `symbols` string and one node-level `exchange`
applied to all of them, so every symbol in one node must share an exchange.

→ `{status, results: [{symbol, exchange, data: {...}}]}`

Prefer this over several `getQuote` nodes: quotes are **not** cached and some
brokers allow only ~1 quote request per second.

### `getDepth`: Get Depth

→ `{status, data: {bids: [{price, quantity}], asks: [...], ltp, totalbuyqty, totalsellqty}}`

### `history`: Historical Data

| Field | Notes |
| --- | --- |
| `symbol`, `exchange` | |
| `interval` | A fixed dropdown on this node: `1m`, `5m`, `15m`, `1h`, `1d`. Editor default `1d`. |
| `days` | Calendar days back. Default 30, editor maximum 365. |
| `startDate`, `endDate` | An explicit range. When both are set they **take precedence over `days`**. |

→ `{status, data: [{timestamp, open, high, low, close, volume, oi}]}`

Timestamps are **epoch seconds**, not ISO strings.

The result is capped at 200 bars by `FLOW_MAX_HISTORY_BARS`, which is an
environment default rather than a fixed ceiling. This node has **no `source`
field**, so it always calls the broker; use `indicator`, `barOffset`, or
`priorPeriodOhlc` when you need `source: "db"`.

### `priorPeriodOhlc`: Prior Period OHLC

The completed previous day/week/month, the reliable way to get PDH/PDL
without off-by-one errors from today's partial candle.

| Field | Values |
| --- | --- |
| `symbol`, `exchange` | |
| `period` | `previous_hour`, `previous_day`, `previous_week`, `previous_month`. Default `previous_day`. |
| `source` | `api` (default) or `db` (Historify) |

→ `{status, symbol, exchange, period, date, open, high, low, close, volume, pdh, pdl, pdc}`

### `barOffset`: Bar Offset

One specific bar counted back from the latest, "the close 20 hours ago", "the
high 3 bars ago".

| Field | Values |
| --- | --- |
| `symbol`, `exchange` | |
| `interval` | Free text. Default `D`. |
| `offsetBars` | 0 is the most recent **closed** bar; 1 is the one before it. |
| `source` | `api` (default) or `db` (Historify) |

Because it counts bars rather than calendar time, the same node answers "20
hours ago" with `interval: "1h", offsetBars: 20`.

→ `{status, symbol, exchange, offsetBars, timestamp, open, high, low, close, volume}`

### `indicator`: Technical Indicator

All 116 single-symbol indicators. Fully covered in
[Indicators](indicators.md).

| Field | Notes |
| --- | --- |
| `symbol`, `exchange` | Required, alongside `indicatorName`. |
| `indicatorName` | e.g. `ema`, `rsi`, `macd`, `supertrend`. Not validated until the run. |
| `params` | The indicator's own parameters **as a JSON string**, e.g. `"{\"period\": 20}"`. A real JSON object here is rejected at import. |
| `interval` | Free text. Default `D`. |
| `source` | `api` (default) or `db` (Historify) |
| `lookbackBars` / `tailBars` | How much history to compute over, and how long the returned `series` is. |
| `offsetBars` | Which past value `at_offset` refers to. |
| `sourceSeries` | Feed another indicator's output in. This is how nesting works. |
| `sourceField` | **Only used when `sourceSeries` is set**, to pick a key out of each upstream row. Blank means the first of `value`, `out0`, `close`. It has no effect on an ordinary indicator node. |

→ `{status, indicator, nested, inputs, params, outputs, latest, previous, at_offset, series, offset_bars, bars_used}`

Single-output indicators expose `value`; multi-output expose `out0`, `out1`, …

### `strategyPnl`: Strategy P&L

Realized, unrealized and total P&L for **one strategy**, not the whole
account, so a workflow can exit on its own performance even when other
strategies hold the same contract.

| Field | Notes |
| --- | --- |
| `strategy` | Defaults to this workflow's name, which is also the tag its order nodes apply. Usually leave blank. |

→ `{status, strategy, realized, today_realized, unrealized, total, today_total, open_quantity, unpriced_legs, legs: [{symbol, exchange, product, quantity, average_price, ltp, realized, today_realized, unrealized}]}`

`total` is realized + unrealized across all sessions; `today_total` is the
intraday equivalent, pairing `today_realized` with the same unrealized figure.

`unpriced_legs` counts open legs with no live price; those are **excluded**
from `unrealized`. A non-zero value means the total is incomplete, guard on
it before acting.

The node returns an **error**, never a zero, when the figures are unknown,
whether the position book is unavailable or the strategy book itself could not
be read. A P&L of zero always means "flat", never "could not tell".

### `openPosition`: Open Position

`symbol`, `exchange`, `product` → `{status, quantity, pnl}`. `quantity` is
signed: negative is short. `pnl` is what `positionCheck`'s `pnl_above` and
`pnl_below` read.

### `calendar`: Calendar

Trading-day facts for a date, and the stateless answer to **"has a new day,
week, month, quarter or year started"**.

| Field | Notes |
| --- | --- |
| `date` | `YYYY-MM-DD`. Blank uses the current trading session date. |

→ `{status, date, is_trading_day, is_trading_holiday, is_weekend, is_special_session, weekday, weekday_num, day, month, quarter, year, week_of_year, day_of_year, is_new_day, is_new_week, is_new_month, is_new_quarter, is_new_year, is_last_day_of_week, is_last_day_of_month, is_last_day_of_quarter, is_last_day_of_year, prev_trading_day, next_trading_day, first_trading_day_of_week, first_trading_day_of_month, first_trading_day_of_quarter, last_trading_day_of_week, last_trading_day_of_month, last_trading_day_of_quarter}`

`is_special_session` distinguishes a Muhurat or other special session from an
ordinary trading day.

Flow keeps no state between runs, so a workflow cannot remember the last run's
date. It does not need to: "a new month started" is the same statement as
"today is the first trading day of this month", which the exchange calendar
answers on its own.

That is also **more correct** than the tests you would otherwise write:

| Naive test | Case it gets wrong |
| --- | --- |
| `{{day}} == 1` | 1 Aug 2026 is a Saturday, so the month opens on the 3rd |
| `{{weekday}} == Monday` | 26 Jan 2026 is Republic Day, so that week opens on Tuesday the 27th |

`is_trading_holiday` is distinct from `is_weekend`, so you can tell a closed
weekday from a weekend. Use `is_last_day_of_month` for month-end square-off.

**Not exchange-aware.** A date is a trading holiday if the exchange calendar
lists one; MCX differs from NSE on a few days a year.

### `intervals`: Supported Intervals

→ `{status, data: {seconds, minutes, hours, days, weeks, months}}`

Use it to confirm your broker supports an interval before relying on it. Not
every broker supports every timeframe.

---

## Symbols, expiries and option chains

### `symbol`: Symbol Info

→ `{status, data: {symbol, brsymbol, lotsize, tick_size, expiry, strike, token, ...}}`

`data.lotsize` is how you size an F&O order correctly.

### `expiry`: Expiry Dates

`symbol`, `exchange`, `instrumenttype` (`options` / `futures`)
→ `{status, message, data: ["04-AUG-26", ...]}`, nearest first.

### `optionSymbol`: Option Symbol

Resolves an ATM-relative strike to a tradable symbol without placing anything.

`underlying`, `exchange`, `expiryDate`, `offset`, `optionType`
→ `{status, symbol, exchange, lotsize, tick_size, freeze_qty, underlying_ltp}`

Note this shape is **flat**, `{{os.symbol}}`, not `{{os.data.symbol}}`.

### `optionChain`: Option Chain

`underlying`, `exchange`, `expiryDate`, `strikeCount`
→ `{status, underlying, underlying_ltp, expiry_date, atm_strike, chain: [{strike, ce: {...}, pe: {...}}]}`

### `syntheticFuture`: Synthetic Future

→ `{status, underlying, expiry, atm_strike, synthetic_future_price, underlying_ltp}`

---

## Account and orders

### `funds`: Funds

→ `{status, data: {availablecash, collateral, m2mrealized, m2munrealized, utiliseddebits}}`

### `orderBook`: Order Book

→ `{status, data: {orders: [...], statistics: {...}}}`

`data.statistics.total_buy_orders` is the usual stateless "have I already
entered today" guard, the order book resets daily and keeps the record after
a position closes.

### `tradeBook`: Trade Book

→ `{status, data: [{tradeid, orderid, symbol, average_price, ...}]}`

### `positionBook`: Position Book

→ `{status, data: [{symbol, exchange, product, quantity, average_price, ltp, pnl}]}`

There is **no `total_pnl` key**: `{{pb.total_pnl}}` resolves to its own
literal text. Sum `data[].pnl` yourself, or use `strategyPnl`.

Account-wide and netted per `(symbol, exchange, product)`, it cannot tell you
which strategy opened a position. Use `strategyPnl` for that.

### `holdings`: Holdings

→ `{status, data: {holdings: [...], statistics: {...}}}`

Note the nesting: `{{hd.data.holdings[0].symbol}}`, not `{{hd.data[0].symbol}}`.

### `margin`: Margin Calculator

Pre-trade margin for a proposed order or basket.

| Field | Notes |
| --- | --- |
| `positionsJson` | A JSON **string** describing the legs. This is the only field the editor exposes, and it is required: a blank value after interpolation is a hard error, not a fallback. |

The single-order fields (`symbol`, `exchange`, `action`, `quantity`,
`product`, `priceType`, `price`) are read by the executor if present in the
JSON, but the panel offers no controls for them, so write them into
`positionsJson`. The legacy field name `positions` is still accepted as a
fallback.

### `getOrderStatus`: Order Status

| Field | Notes |
| --- | --- |
| `orderId` | Required. |

→ `{status, data: {order_status, average_price, quantity, ...}}`

The node calls the broker once and returns immediately. `waitForCompletion`
appears in older exports and is read by nothing: there is no polling and no
wait. To react to a fill, use `orderUpdateTrigger` in a separate workflow.

### `holidays` / `timings`

`holidays` takes a `year` → the exchange holiday list. `timings` takes a
`date` → market open/close for that day. Use them to avoid firing on a
holiday.

---

## Streaming

These maintain a WebSocket subscription and pass the latest tick to their
output variable. If WebSocket is unavailable they fall back to a single REST
call, identical from the workflow's point of view.

All three write a wrapped envelope to their output variable, not the bare
value. Read through the envelope.

| Node | Output variable holds | Read the price as |
| --- | --- | --- |
| `subscribeLtp` | `{status, type: "ltp", symbol, exchange, ltp, source}` | `{{lt.ltp}}` |
| `subscribeQuote` | `{status, type, symbol, exchange, data: {ltp, open, high, low, close, volume, ...}, source}` | `{{q.data.ltp}}` |
| `subscribeDepth` | `{status, type, symbol, exchange, data: {bids, asks, totalbuyqty, totalsellqty, ltp}, source}` | `{{d.data.bids[0].price}}` |
| `unsubscribe` | `streamType`: `ltp` / `quote` / `depth` / `all`. Empty `symbol` = all symbols. | |

`{{lt}}` used as a number and `{{q.ltp}}` are the two mistakes this shape
causes. Neither raises: the unresolved path is passed through as literal
text.

---

## Utilities

### `variable`: Variable

| Field | Values |
| --- | --- |
| `variableName` | |
| `operation` | `set`, `add`, `increment`, `decrement` |
| `value` | Literal or `{{path}}`. For `set`, a string starting `{` or `[` is parsed as JSON. |

**Only those four operations are implemented.** The editor's dropdown also
offers `get`, `subtract`, `multiply`, `divide`, `parse_json`, `stringify`, and
`append`, and so do the `sourceVariable` and `jsonPath` fields that accompany
them. None of the seven has an executor branch: the node returns
`status: "success"` and writes nothing. There is no `extract` operation at
all. Use `mathExpression` for arithmetic beyond `add` / `increment` /
`decrement`, and read nested values with a `{{path}}` directly.

**Variables do not survive the run.** An `increment` counts within one
execution only; it is reset on the next tick. Anything that must persist has
to come from the broker.

### `mathExpression`: Math Expression

`expression`, arithmetic over interpolated values, e.g.
`"{{q.data.ltp}} * 1.02"`. Result goes to `outputVariable`.

**Arithmetic only.** The supported operators are `+`, `-`, `*`, `/`, `%`, and
`**`, plus unary minus and plus. A comparison such as `{{a}} > {{b}}` raises
"Unsupported expression type: Compare"; use `varCondition` for comparisons.
There is no string manipulation, no date arithmetic, and no function calls.
Exponents are capped at 100.

### `log`: Log

`message` (interpolated) and `level` (`info`, `warn`, `error`). The value is
`warn`, not `warning`: anything else is stored verbatim and rendered at info
severity. Written to the workflow's execution log, the primary way to debug a
run.

### `telegramAlert`: Telegram Alert

`message` (interpolated). Requires the Telegram bot to be configured and the
workflow owner's account linked to it.

The panel still shows an "OpenAlgo Username" box, but the executor never reads
it. The recipient is resolved from the workflow's own API key, so the alert
always reaches the workflow owner regardless of what is typed there.

### `whatsappAlert`: WhatsApp Alert

`to` and `message`. Requires the WhatsApp bot to be paired.

### `delay`: Delay

`delayValue` + `delayUnit` (`seconds` / `minutes` / `hours`), or `delayMs`.

**Blocking, and silently capped at 300 seconds.** A longer value logs a
warning and waits 300 seconds instead, then continues: the run does not fail,
it just proceeds early. A node configured as "2 hours" waits five minutes.

The run sleeps and holds its execution slot for whatever it does wait. Do not
use it to pace a trading session; use a second workflow on its own schedule.

### `waitUntil`: Wait Until

`targetTime` (`"HH:MM"`). Returns immediately with `{status, waited: false}`
if the time has already passed today.

`waitUntil` is genuinely uncapped, unlike `delay`. Set to `15:20` from a
09:20 run it blocks that run for six hours, holding the workflow lock the
whole time so no other tick of the same workflow can start. Prefer a
scheduled trigger.

### `group`: Group

Visual grouping only. No execution behaviour.

---

## Node count by category

| Category | Count | Nodes |
| --- | --- | --- |
| Triggers | 4 | `start`, `priceAlert`, `webhookTrigger`, `orderUpdateTrigger` |
| Order placement | 10 | `placeOrder`, `smartOrder`, `optionsOrder`, `optionsMultiOrder`, `basketOrder`, `splitOrder`, `modifyOrder`, `cancelOrder`, `cancelAllOrders`, `closePositions` |
| Conditions and logic | 9 | `priceCondition`, `varCondition`, `timeWindow`, `timeCondition`, `positionCheck`, `fundCheck`, `andGate`, `orGate`, `notGate` |
| Market data | 11 | `getQuote`, `multiQuotes`, `getDepth`, `history`, `priorPeriodOhlc`, `barOffset`, `indicator`, `strategyPnl`, `openPosition`, `intervals`, `calendar` |
| Symbols and options | 5 | `symbol`, `expiry`, `optionSymbol`, `optionChain`, `syntheticFuture` |
| Account and orders | 9 | `funds`, `orderBook`, `tradeBook`, `positionBook`, `holdings`, `margin`, `getOrderStatus`, `holidays`, `timings` |
| Streaming | 4 | `subscribeLtp`, `subscribeQuote`, `subscribeDepth`, `unsubscribe` |
| Utilities | 9 | `variable`, `mathExpression`, `log`, `telegramAlert`, `whatsappAlert`, `delay`, `waitUntil`, `httpRequest`, `group` |
| **Total** | **61** | |

The sections above group nodes by task. The editor's palette groups them
slightly differently: `calendar`, `holidays`, and `timings` sit under
Utilities there, and `holdings`, `funds`, and `margin` under Risk Management.

For the exact JSON schema of every field, the format an AI needs to generate
an importable workflow, see
[`docs/prompt/flow-import-format.md`](https://github.com/marketcalls/openalgo/blob/main/docs/prompt/flow-import-format.md)
in the main repository.
