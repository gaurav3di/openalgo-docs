# Tutorials

Ten complete strategies, from a first workflow to calendar-driven rebalancing.

**Every JSON on this page was executed against a running OpenAlgo instance
with a live broker connection in Analyzer mode.** The log output shown is
real output from that run, not illustration.

## How to run these

1. Open `/flow`, click **Import**
2. Paste the JSON (or save it as a `.json` file and use the file picker)
3. Click **Save**, then **Run** to execute once
4. Read the execution log panel, every node reports what it did

Turn **Analyzer mode on** first. Orders will be simulated.

---

## 1. Quotes, maths, and variables

The smallest useful workflow: read a quote, compute the percentage move, log
it. This teaches `outputVariable` and `{{...}}` interpolation.

```json
{
  "name": "T1 quote and variables",
  "nodes": [
    { "id": "n1", "type": "start", "position": { "x": 0, "y": 0 },
      "data": { "scheduleType": "daily", "time": "09:20", "days": [0,1,2,3,4], "marketHoursOnly": true } },
    { "id": "q", "type": "getQuote", "position": { "x": 0, "y": 100 },
      "data": { "symbol": "RELIANCE", "exchange": "NSE", "outputVariable": "q" } },
    { "id": "m", "type": "mathExpression", "position": { "x": 0, "y": 200 },
      "data": { "expression": "({{q.data.ltp}} - {{q.data.prev_close}}) / {{q.data.prev_close}} * 100", "outputVariable": "chgPct" } },
    { "id": "l", "type": "log", "position": { "x": 0, "y": 300 },
      "data": { "message": "RELIANCE ltp={{q.data.ltp}} open={{q.data.open}} prevClose={{q.data.prev_close}} change={{chgPct}}%", "level": "info" } }
  ],
  "edges": [
    { "id": "e1", "source": "n1", "target": "q" },
    { "id": "e2", "source": "q", "target": "m" },
    { "id": "e3", "source": "m", "target": "l" }
  ]
}
```

**Key point.** `mathExpression` supports `+ - * / % **` and parentheses over
interpolated values. It cannot do date arithmetic or call functions.

---

## 2. Indicator conditions and branching

Compute RSI, then take a different path depending on its value.

```json
{
  "name": "T2 RSI check",
  "nodes": [
    { "id": "n1", "type": "start", "position": { "x": 0, "y": 0 },
      "data": { "scheduleType": "interval", "intervalValue": 5, "intervalUnit": "minutes", "marketHoursOnly": true } },
    { "id": "r", "type": "indicator", "position": { "x": 0, "y": 100 },
      "data": { "symbol": "RELIANCE", "exchange": "NSE", "interval": "D", "source": "api",
                "indicatorName": "rsi", "params": "{\"period\":14}",
                "lookbackBars": 100, "tailBars": 5, "offsetBars": 0, "outputVariable": "rsi" } },
    { "id": "c", "type": "varCondition", "position": { "x": 0, "y": 200 },
      "data": { "leftValue": "{{rsi.latest.value}}", "operator": "<", "rightValue": "70" } },
    { "id": "y", "type": "log", "position": { "x": 0, "y": 300 },
      "data": { "message": "RSI {{rsi.latest.value}} below 70 - entry allowed", "level": "info" } },
    { "id": "n", "type": "log", "position": { "x": 250, "y": 300 },
      "data": { "message": "RSI {{rsi.latest.value}} too hot - skip", "level": "info" } }
  ],
  "edges": [
    { "id": "e1", "source": "n1", "target": "r" },
    { "id": "e2", "source": "r", "target": "c" },
    { "id": "e3", "source": "c", "sourceHandle": "true",  "target": "y" },
    { "id": "e4", "source": "c", "sourceHandle": "false", "target": "n" }
  ]
}
```

Verified output:

```
Computing rsi for RELIANCE (D, 100 bars)
rsi latest: {'value': 43.385612}
Var check: 43.385612 < 70.0 = True
[LOG] RSI 43.385612 below 70 - entry allowed
```

**Key point.** `varCondition` compares any two interpolated values. Unlike
`priceCondition` (which always re-fetches a live quote field), it works on
indicator outputs, previous-period levels, and literals. If an operand does
not resolve to a number it **refuses to evaluate and takes neither branch**:
a typo cannot silently route your else-path into a trade.

---

## 3. Crossovers

There is no crossover node, `crossover` needs two series. Build it from two
indicator nodes and an `andGate`.

A golden cross means: fast is above slow **now**, and fast was at or below
slow **on the previous bar**.

```json
{
  "name": "T3 EMA golden cross",
  "nodes": [
    { "id": "n1", "type": "start", "position": { "x": 0, "y": 0 },
      "data": { "scheduleType": "once", "executeAt": "2099-01-01", "marketHoursOnly": false } },
    { "id": "f", "type": "indicator", "position": { "x": 0, "y": 100 },
      "data": { "symbol": "NIFTY", "exchange": "NSE_INDEX", "interval": "D", "source": "api",
                "indicatorName": "ema", "params": "{\"period\":9}",
                "lookbackBars": 120, "tailBars": 3, "outputVariable": "fast" } },
    { "id": "s", "type": "indicator", "position": { "x": 0, "y": 200 },
      "data": { "symbol": "NIFTY", "exchange": "NSE_INDEX", "interval": "D", "source": "api",
                "indicatorName": "ema", "params": "{\"period\":21}",
                "lookbackBars": 120, "tailBars": 3, "outputVariable": "slow" } },
    { "id": "c1", "type": "varCondition", "position": { "x": 0, "y": 300 },
      "data": { "leftValue": "{{fast.latest.value}}", "operator": ">", "rightValue": "{{slow.latest.value}}" } },
    { "id": "c2", "type": "varCondition", "position": { "x": 250, "y": 300 },
      "data": { "leftValue": "{{fast.previous.value}}", "operator": "<=", "rightValue": "{{slow.previous.value}}" } },
    { "id": "and", "type": "andGate", "position": { "x": 120, "y": 400 }, "data": { "inputCount": 2 } },
    { "id": "log", "type": "log", "position": { "x": 120, "y": 500 },
      "data": { "message": "GOLDEN CROSS: fast={{fast.latest.value}} slow={{slow.latest.value}} (prev fast={{fast.previous.value}} slow={{slow.previous.value}})", "level": "info" } }
  ],
  "edges": [
    { "id": "e1", "source": "n1", "target": "f" },
    { "id": "e2", "source": "f", "target": "s" },
    { "id": "e3", "source": "s", "target": "c1" },
    { "id": "e4", "source": "s", "target": "c2" },
    { "id": "e5", "source": "c1", "target": "and", "targetHandle": "input-0" },
    { "id": "e6", "source": "c2", "target": "and", "targetHandle": "input-1" },
    { "id": "e7", "source": "and", "sourceHandle": "true", "target": "log" }
  ]
}
```

Verified output, a genuine EMA9/EMA21 cross on NIFTY daily:

```
ema latest: {'value': 24055.354181}     <- fast
ema latest: {'value': 24045.22283}      <- slow
Var check: 24055.354181 > 24045.22283 = True
andGate: waiting for 1 more input(s) before evaluating
Var check: 24008.692726 <= 24025.545113 = True
AND Gate: [True, True] -> True
[LOG] GOLDEN CROSS: fast=24055.354181 slow=24045.22283 ...
```

**For a death cross**, flip both operators: `<` on latest, `>=` on previous.

**To detect a cross on an earlier bar**, add `offsetBars` to both indicators
and compare `{{fast.at_offset.value}}` against `{{slow.at_offset.value}}`.

---

## 4. Multi-timeframe filter

Trade only when the daily trend and the 15-minute momentum agree.

```json
{
  "name": "T4 multi-timeframe filter",
  "nodes": [
    { "id": "n1", "type": "start", "position": { "x": 0, "y": 0 },
      "data": { "scheduleType": "interval", "intervalValue": 15, "intervalUnit": "minutes", "marketHoursOnly": true } },
    { "id": "d", "type": "indicator", "position": { "x": 0, "y": 100 },
      "data": { "symbol": "NIFTY", "exchange": "NSE_INDEX", "interval": "D", "source": "api",
                "indicatorName": "ema", "params": "{\"period\":20}", "lookbackBars": 100, "tailBars": 3, "outputVariable": "emaD" } },
    { "id": "i", "type": "indicator", "position": { "x": 0, "y": 200 },
      "data": { "symbol": "NIFTY", "exchange": "NSE_INDEX", "interval": "15m", "source": "api",
                "indicatorName": "rsi", "params": "{\"period\":14}", "lookbackBars": 100, "tailBars": 3, "outputVariable": "rsi15" } },
    { "id": "q", "type": "getQuote", "position": { "x": 0, "y": 300 },
      "data": { "symbol": "NIFTY", "exchange": "NSE_INDEX", "outputVariable": "q" } },
    { "id": "c1", "type": "varCondition", "position": { "x": 0, "y": 400 },
      "data": { "leftValue": "{{q.data.ltp}}", "operator": ">", "rightValue": "{{emaD.latest.value}}" } },
    { "id": "c2", "type": "varCondition", "position": { "x": 250, "y": 400 },
      "data": { "leftValue": "{{rsi15.latest.value}}", "operator": ">", "rightValue": "50" } },
    { "id": "g", "type": "andGate", "position": { "x": 120, "y": 500 }, "data": { "inputCount": 2 } },
    { "id": "y", "type": "log", "position": { "x": 0, "y": 600 },
      "data": { "message": "Daily uptrend + 15m momentum: ltp={{q.data.ltp}} emaD={{emaD.latest.value}} rsi15={{rsi15.latest.value}}", "level": "info" } },
    { "id": "n", "type": "log", "position": { "x": 260, "y": 600 },
      "data": { "message": "Filter not aligned - no trade", "level": "info" } }
  ],
  "edges": [
    { "id": "e1", "source": "n1", "target": "d" },
    { "id": "e2", "source": "d", "target": "i" },
    { "id": "e3", "source": "i", "target": "q" },
    { "id": "e4", "source": "q", "target": "c1" },
    { "id": "e5", "source": "q", "target": "c2" },
    { "id": "e6", "source": "c1", "target": "g", "targetHandle": "input-0" },
    { "id": "e7", "source": "c2", "target": "g", "targetHandle": "input-1" },
    { "id": "e8", "source": "g", "sourceHandle": "true",  "target": "y" },
    { "id": "e9", "source": "g", "sourceHandle": "false", "target": "n" }
  ]
}
```

Verified output:

```
ema latest: {'value': 24047.893645}     <- daily EMA20
rsi latest: {'value': 60.174092}        <- 15-minute RSI
Var check: 24250.2 > 24047.893645 = True
AND Gate: [True, True] -> True
[LOG] Daily uptrend + 15m momentum: ltp=24250.2 emaD=24047.893645 rsi15=60.174092
```

**Key point.** Two indicator nodes on the same symbol at *different*
intervals are two distinct fetches, the cache only collapses identical
requests. That is correct and unavoidable.

---

## 5. Previous-day breakout with a gap filter

The classic PDH breakout, with the refinement that a gap-up open should not
trade until price has come back to test the level.

Flow has no memory between runs, so "price returned to PDH earlier today"
cannot be a stored flag. Instead read **today's session low** from the quote:
if the day's low is at or below PDH, price has already visited that level
today. That makes the filter entirely stateless.

```json
{
  "name": "T5 PDH breakout with gap-up retest filter",
  "nodes": [
    { "id": "n1", "type": "start", "position": { "x": 0, "y": 0 },
      "data": { "scheduleType": "interval", "intervalValue": 1, "intervalUnit": "minutes", "marketHoursOnly": true } },
    { "id": "pd", "type": "priorPeriodOhlc", "position": { "x": 0, "y": 100 },
      "data": { "symbol": "NIFTY", "exchange": "NSE_INDEX", "period": "previous_day", "source": "api", "outputVariable": "pd" } },
    { "id": "q", "type": "getQuote", "position": { "x": 0, "y": 200 },
      "data": { "symbol": "NIFTY", "exchange": "NSE_INDEX", "outputVariable": "q" } },
    { "id": "win", "type": "timeWindow", "position": { "x": 0, "y": 300 },
      "data": { "startTime": "09:20", "endTime": "15:00" } },
    { "id": "brk", "type": "varCondition", "position": { "x": 0, "y": 400 },
      "data": { "leftValue": "{{q.data.ltp}}", "operator": ">", "rightValue": "{{pd.pdh}}" } },
    { "id": "retest", "type": "varCondition", "position": { "x": 250, "y": 400 },
      "data": { "leftValue": "{{q.data.low}}", "operator": "<=", "rightValue": "{{pd.pdh}}" } },
    { "id": "pos", "type": "positionCheck", "position": { "x": 500, "y": 400 },
      "data": { "symbol": "NIFTY", "exchange": "NSE_INDEX", "product": "NRML", "condition": "not_exists" } },
    { "id": "g", "type": "andGate", "position": { "x": 250, "y": 500 }, "data": { "inputCount": 4 } },
    { "id": "go", "type": "log", "position": { "x": 150, "y": 600 },
      "data": { "message": "ENTRY: LTP {{q.data.ltp}} broke PDH {{pd.pdh}} (day low {{q.data.low}} confirms retest, PDL {{pd.pdl}})", "level": "info" } },
    { "id": "skip", "type": "log", "position": { "x": 450, "y": 600 },
      "data": { "message": "No entry: ltp={{q.data.ltp}} pdh={{pd.pdh}} dayLow={{q.data.low}}", "level": "info" } }
  ],
  "edges": [
    { "id": "e1", "source": "n1", "target": "pd" },
    { "id": "e2", "source": "pd", "target": "q" },
    { "id": "e3", "source": "q", "target": "win" },
    { "id": "e4", "source": "win", "target": "brk" },
    { "id": "e5", "source": "win", "target": "retest" },
    { "id": "e6", "source": "win", "target": "pos" },
    { "id": "e7",  "source": "win",    "target": "g", "targetHandle": "input-0" },
    { "id": "e8",  "source": "brk",    "target": "g", "targetHandle": "input-1" },
    { "id": "e9",  "source": "retest", "target": "g", "targetHandle": "input-2" },
    { "id": "e10", "source": "pos",    "target": "g", "targetHandle": "input-3" },
    { "id": "e11", "source": "g", "sourceHandle": "true",  "target": "go" },
    { "id": "e12", "source": "g", "sourceHandle": "false", "target": "skip" }
  ]
}
```

Verified output, the filter correctly declined a gap-up that never retested:

```
previous_day OHLC for NIFTY: H=24041.15 L=23954.6 C=23985.35
Var check: 24250.2 > 24041.15 = True        <- above PDH
Var check: 24136.75 <= 24041.15 = False     <- day low never reached PDH
Position check: qty=0 == 0 = True
AND Gate: [False, True, False, True] -> False
[LOG] No entry: ltp=24250.2 pdh=24041.15 dayLow=24136.75
```

**For the PDL short side**, mirror it: `{{q.data.ltp}} < {{pd.pdl}}` and
`{{q.data.high}} >= {{pd.pdl}}`.

**"One trade per breakout"** is enforced by `positionCheck` with
`not_exists`, it asks the broker, so it survives restarts, unlike a counter
variable.

To actually trade it, replace the `go` log node with the options entry from
Tutorial 7.

---

## 6. Historical lookback

Previous day, previous week, a bar five sessions back, and an indicator value
five bars back, all in one workflow.

```json
{
  "name": "T6 historical lookback",
  "nodes": [
    { "id": "n1", "type": "start", "position": { "x": 0, "y": 0 },
      "data": { "scheduleType": "once", "executeAt": "2099-01-01", "marketHoursOnly": false } },
    { "id": "pd", "type": "priorPeriodOhlc", "position": { "x": 0, "y": 100 },
      "data": { "symbol": "NIFTY", "exchange": "NSE_INDEX", "period": "previous_day", "source": "api", "outputVariable": "pday" } },
    { "id": "pw", "type": "priorPeriodOhlc", "position": { "x": 0, "y": 200 },
      "data": { "symbol": "NIFTY", "exchange": "NSE_INDEX", "period": "previous_week", "source": "api", "outputVariable": "pweek" } },
    { "id": "b5", "type": "barOffset", "position": { "x": 0, "y": 300 },
      "data": { "symbol": "NIFTY", "exchange": "NSE_INDEX", "interval": "D", "source": "api", "offsetBars": 5, "outputVariable": "bar5" } },
    { "id": "st", "type": "indicator", "position": { "x": 0, "y": 400 },
      "data": { "symbol": "NIFTY", "exchange": "NSE_INDEX", "interval": "D", "source": "api",
                "indicatorName": "supertrend", "params": "{\"period\":10,\"multiplier\":3}",
                "lookbackBars": 150, "tailBars": 10, "offsetBars": 5, "outputVariable": "st" } },
    { "id": "l", "type": "log", "position": { "x": 0, "y": 500 },
      "data": { "message": "PDH={{pday.pdh}} PDL={{pday.pdl}} | prevWeekH={{pweek.pdh}} prevWeekL={{pweek.pdl}} | close5BarsAgo={{bar5.close}} | supertrend now={{st.latest.out0}} 5barsAgo={{st.at_offset.out0}} dir={{st.latest.out1}}", "level": "info" } }
  ],
  "edges": [
    { "id": "e1", "source": "n1", "target": "pd" },
    { "id": "e2", "source": "pd", "target": "pw" },
    { "id": "e3", "source": "pw", "target": "b5" },
    { "id": "e4", "source": "b5", "target": "st" },
    { "id": "e5", "source": "st", "target": "l" }
  ]
}
```

**Key point.** `offsetBars: 5` on a daily chart is five *trading* bars, which
spans seven calendar days across a weekend. Supertrend legitimately holds a
flat level through a sustained trend, so identical values at different
offsets are often correct, not a bug, check `out1` (direction) to confirm.

---

## 7. Risk-guarded ATM options entry

Two guards, available funds and no existing position, before buying a
current-week ATM call.

```json
{
  "name": "T7 risk-guarded ATM option entry",
  "nodes": [
    { "id": "n1", "type": "start", "position": { "x": 0, "y": 0 },
      "data": { "scheduleType": "once", "executeAt": "2099-01-01", "marketHoursOnly": false } },
    { "id": "f", "type": "fundCheck", "position": { "x": 0, "y": 100 }, "data": { "minAvailable": 1000 } },
    { "id": "p", "type": "positionCheck", "position": { "x": 250, "y": 100 },
      "data": { "symbol": "NIFTY", "exchange": "NSE_INDEX", "product": "NRML", "condition": "not_exists" } },
    { "id": "g", "type": "andGate", "position": { "x": 120, "y": 200 }, "data": { "inputCount": 2 } },
    { "id": "o", "type": "optionsOrder", "position": { "x": 0, "y": 300 },
      "data": { "underlying": "NIFTY", "expiryType": "current_week", "offset": "ATM", "optionType": "CE",
                "action": "BUY", "quantity": 1, "priceType": "MARKET", "product": "NRML", "outputVariable": "ce" } },
    { "id": "l", "type": "log", "position": { "x": 0, "y": 400 },
      "data": { "message": "Placed ATM CE order id={{ce.orderid}}", "level": "info" } },
    { "id": "s", "type": "log", "position": { "x": 300, "y": 300 },
      "data": { "message": "Guard blocked entry", "level": "info" } }
  ],
  "edges": [
    { "id": "e1", "source": "n1", "target": "f" },
    { "id": "e2", "source": "n1", "target": "p" },
    { "id": "e3", "source": "f", "target": "g", "targetHandle": "input-0" },
    { "id": "e4", "source": "p", "target": "g", "targetHandle": "input-1" },
    { "id": "e5", "source": "g", "sourceHandle": "true", "target": "o" },
    { "id": "e6", "source": "o", "target": "l" },
    { "id": "e7", "source": "g", "sourceHandle": "false", "target": "s" }
  ]
}
```

Verified output (Analyzer mode):

```
Fund check: available=9984452.0 >= 1000.0 = True
andGate: waiting for 1 more input(s) before evaluating
Position check: qty=0 == 0 = True
AND Gate: [True, True] -> True
Resolved expiry: current_week -> 04AUG26
Options order result: {'status': 'success', 'orderid': '26072924089946',
  'symbol': 'NIFTY04AUG2624250CE', 'exchange': 'NFO', 'offset': 'ATM',
  'option_type': 'CE', 'mode': 'analyze'}
```

**Key points.**

* `optionsOrder` resolves the expiry and strike for you: you never hand-write
  `NIFTY04AUG2624250CE`.
* `quantity` here is **in lots**. `placeOrder`, `smartOrder`, `splitOrder`,
  and `basketOrder` take quantity in **shares**. This asymmetry is the single
  most common sizing mistake.
* `expiryType` accepts `current_week`, `next_week`, `current_month`,
  `next_month`.
* `offset` accepts `ATM`, `ITM1`-`ITM50`, `OTM1`-`OTM50`.

---

## 8. Reacting to a fill

`orderUpdateTrigger` fires the moment a matching order changes status, no
polling loop.

```json
{
  "name": "T8 react to a fill",
  "nodes": [
    { "id": "n1", "type": "orderUpdateTrigger", "position": { "x": 0, "y": 0 },
      "data": { "symbol": "NIFTY04AUG2624250CE", "exchange": "NFO", "status": "complete", "trigger": "once" } },
    { "id": "a", "type": "telegramAlert", "position": { "x": 0, "y": 100 },
      "data": { "username": "your_openalgo_login", "message": "FILLED {{webhook.symbol}} qty={{webhook.filled_quantity}} avg={{webhook.average_price}}" } }
  ],
  "edges": [ { "id": "e1", "source": "n1", "target": "a" } ]
}
```

The event payload is exposed as `{{webhook.*}}`: `orderid`, `symbol`,
`exchange`, `order_status`, `filled_quantity`, `average_price`,
`rejection_reason`.

**Key points.**

* Set at least one of Order ID or Symbol. An unfiltered watch would fire on
  every order in the account and is rejected.
* Order ID must be a **literal** broker order id, a trigger has no upstream
  node, so `{{ce.orderid}}` cannot be resolved and is rejected with a 400.
  Filter by symbol instead.
* `status` accepts `any`, `open`, `trigger pending`, `complete`, `rejected`,
  `cancelled`.
* `trigger: "once"` stops watching after the first match; `"every_time"`
  keeps watching.
* Watches survive a restart, they are restored from active workflows on boot.

---

## 9. Exit on this strategy's own P&L

The account's position book nets everything and carries no strategy label, so
"am I up 5000 on *this* strategy" cannot be answered from `positionBook` when
another strategy holds the same contract. `strategyPnl` answers it.

```json
{
  "name": "T9 strategy pnl exit",
  "nodes": [
    { "id": "n1", "type": "start", "position": { "x": 0, "y": 0 },
      "data": { "scheduleType": "interval", "intervalValue": 1, "intervalUnit": "minutes", "marketHoursOnly": true } },
    { "id": "pnl", "type": "strategyPnl", "position": { "x": 0, "y": 100 },
      "data": { "outputVariable": "pnl" } },
    { "id": "hasqty", "type": "varCondition", "position": { "x": 0, "y": 200 },
      "data": { "leftValue": "{{pnl.open_quantity}}", "operator": "!=", "rightValue": "0" } },
    { "id": "target", "type": "varCondition", "position": { "x": 0, "y": 300 },
      "data": { "leftValue": "{{pnl.total}}", "operator": ">=", "rightValue": "5000" } },
    { "id": "flat", "type": "closePositions", "position": { "x": 0, "y": 400 }, "data": {} },
    { "id": "say", "type": "log", "position": { "x": 220, "y": 400 },
      "data": { "message": "Target hit: total={{pnl.total}} realized={{pnl.realized}} unrealized={{pnl.unrealized}}", "level": "info" } }
  ],
  "edges": [
    { "id": "e1", "source": "n1", "target": "pnl" },
    { "id": "e2", "source": "pnl", "target": "hasqty" },
    { "id": "e3", "source": "hasqty", "target": "target", "sourceHandle": "true" },
    { "id": "e4", "source": "target", "target": "flat", "sourceHandle": "true" },
    { "id": "e5", "source": "target", "target": "say", "sourceHandle": "true" }
  ]
}
```

Verified with one open leg of 75 NIFTY CE bought at 100 and marked at 170:

```
realized=0.0  unrealized=5250.0  total=5250.0  openQty=75.0  unpriced_legs=0
-> total >= 5000, exit branch taken
```

**Key points.**

* Leave `strategy` blank. It defaults to the workflow's own name, which is the
  same tag this workflow's order nodes apply, so entry and exit agree with no
  configuration.
* The `open_quantity != 0` guard first. Without it the workflow re-fires
  `closePositions` every minute after the position is already flat, because
  realized P&L stays above the target for the rest of the session.
* `total` is realized + unrealized. Use `today_realized` for an intraday-only
  target that ignores P&L carried in from previous sessions.
* Swap `>=` for `<=` and a negative number to get a stop-loss instead.
* **`closePositions` is account-wide**, not strategy-scoped. If several
  strategies hold positions at once, close the specific leg with a
  `placeOrder` or `smartOrder` (`positionSize: 0`) instead.

**Before relying on this.** The strategy book is built from orders placed
**through OpenAlgo with a strategy tag**, Flow nodes and `/api/v1/` calls
that carry `strategy`. A position you opened by hand in the broker terminal is
invisible to it, so its P&L is not counted. Check `unpriced_legs` too: it
counts open legs with no live price, which are excluded from `unrealized`, so
a non-zero value means `total` is understated.

---

## 10. Run only on the first trading day of the month

A monthly rebalance must fire once, on the first day the exchange actually
trades. `{{day}} == 1` gets this wrong whenever the 1st is a weekend or a
holiday. The `calendar` node answers it directly.

```json
{
  "name": "T10 monthly rebalance",
  "nodes": [
    { "id": "n1", "type": "start", "position": { "x": 0, "y": 0 },
      "data": { "scheduleType": "daily", "time": "09:20", "days": [0,1,2,3,4], "marketHoursOnly": true } },
    { "id": "cal", "type": "calendar", "position": { "x": 0, "y": 100 },
      "data": { "outputVariable": "cal" } },
    { "id": "isnew", "type": "varCondition", "position": { "x": 0, "y": 200 },
      "data": { "leftValue": "{{cal.is_new_month}}", "operator": "==", "rightValue": "true" } },
    { "id": "go", "type": "log", "position": { "x": 0, "y": 300 },
      "data": { "message": "New month opened on {{cal.date}} ({{cal.weekday}}) - rebalancing. Quarter {{cal.quarter}}, week {{cal.week_of_year}}.", "level": "info" } },
    { "id": "skip", "type": "log", "position": { "x": 260, "y": 300 },
      "data": { "message": "{{cal.date}} is not the first trading day of the month - nothing to do", "level": "info" } }
  ],
  "edges": [
    { "id": "e1", "source": "n1", "target": "cal" },
    { "id": "e2", "source": "cal", "target": "isnew" },
    { "id": "e3", "source": "isnew", "sourceHandle": "true",  "target": "go" },
    { "id": "e4", "source": "isnew", "sourceHandle": "false", "target": "skip" }
  ]
}
```

**Key points.**

* The schedule runs every weekday; the `calendar` node decides whether today is
  the day. That is deliberate, a monthly schedule could not know which date
  the exchange actually opens on.
* Swap `is_new_month` for `is_new_week`, `is_new_quarter` or `is_new_year`. Use
  `is_last_day_of_month` for a month-end square-off instead.
* Flow keeps no state between runs, so this cannot work by remembering the last
  run. It does not need to: "a new month started" is the same statement as
  "today is the first trading day of this month", and the exchange calendar
  answers that on its own.
* `is_trading_holiday` is distinct from `is_weekend`, so you can log *why* a day
  was skipped.
* Blank `date` uses the current trading session date, which differs from the
  calendar date between midnight and the 03:00 IST rollover.

Real 2026 dates this handles correctly, and the naive tests do not:

| Date | Day | `is_new_month` | `is_new_week` |
| --- | --- | --- | --- |
| 1 Aug | Saturday | false | false |
| 3 Aug | Monday | **true** | **true** |
| 26 Jan (Republic Day) | Monday | false | false |
| 27 Jan | Tuesday | false | **true** |

---

## Where to go next

* [Limitations and Gotchas](limitations.md): read before trading these live
* [Indicators](indicators.md): the full 116-function reference
* [Market Data](market-data.md): timeframes, the 200-bar ceiling, rate limits
