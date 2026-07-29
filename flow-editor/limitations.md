# Limitations and Gotchas

Read this before running a Flow strategy with real money. Most items have a
workaround; a few are hard boundaries.

## Hard limitations

### No state between runs

`variable` values live for one run only. A counter incremented on Monday is
back to zero on Tuesday — and back to zero on the very next run, minutes
later.

**Do not** use a variable to answer "have I already entered today?".

**Instead**, ask the broker, which is the real source of truth:

* `positionCheck` with `condition: "not_exists"` — no open position
* `orderBook` and inspect `{{orders.data.orders[0]...}}`
* `openPosition` for a specific symbol's quantity

This is why [Tutorial 5](tutorials.md#5-previous-day-breakout-with-a-gap-filter)
derives "price retested the level today" from the session low in the quote
rather than from a stored flag.

### No loops or iteration

You cannot iterate over a list of symbols. One workflow handles one symbol.

**Workaround.** Export the workflow, edit the symbol, import it under a new
name. Or drive it from outside: have an external scanner POST to a
`webhookTrigger` with the symbol in the payload, and reference
`{{webhook.symbol}}` in the order node.

### No backtesting

Flow executes forward in real time only. There is no historical replay.

**Workaround.** Prototype the logic in the Python Strategy Host (`/python`)
or with the `openalgo` SDK, which gives you pandas DataFrames and full
backtesting libraries. Port the validated logic into Flow for execution.

### No pandas objects

Flow variables are JSON. `history` returns an array of record objects and
`indicator` returns single values plus a fixed-length array — never a
`pandas.Series`.

You can read individual values (`{{h.data[0].close}}`) and feed an array into
a nested indicator (`sourceSeries`), but you cannot do vectorised series
maths inside Flow.

**Workaround.** The Python Strategy Host gives you real pandas:
`client.history(...)` returns a DataFrame and `df['close']` is a Series.

### Triggers cannot read upstream data

A trigger is the entry point of the graph, so nothing precedes it. Its
configuration fields cannot contain `{{variables}}`.

The practical case: you cannot make an `orderUpdateTrigger` watch
`{{ce.orderid}}` from a previous workflow. Filter by symbol instead.

### Two-series indicators are unavailable

`crossover`, `crossunder`, `cross`, `correlation`, and `beta` need two
independent series. The `indicator` node reads one symbol.

**Workaround.** Build crossovers from two indicator nodes plus an `andGate` —
[Tutorial 3](tutorials.md#3-crossovers). For correlation between two symbols,
use the Python Strategy Host.

### Only single-series indicators can be nested

`sourceSeries` collapses an upstream indicator to one numeric series, so
anything requiring independent high/low/close (`atr`, `supertrend`, `adx`,
`stochastic`, …) cannot be nested on top of it. You get a clear error, not a
wrong number.

`sma`, `ema`, `rsi`, `wma`, `stdev`, `highest`, `lowest`, and friends nest
fine.

---

## Gotchas that cost money

### Quantity units differ by node

| Node | Unit |
| --- | --- |
| `optionsOrder`, `optionsMultiOrder` | **lots** |
| `placeOrder`, `smartOrder`, `splitOrder`, `basketOrder` | **shares** |

`quantity: 1` on an options node is one lot (75 NIFTY units). The same value
on `placeOrder` is one share. This is the most common sizing error.

### Gate wiring determines whether the else-branch works

Feeding a gate through `sourceHandle: "true"` edges means the gate is only
reached when the condition is true — so its `false` branch can never fire.

Use pass-through wiring (`targetHandle` only, no `sourceHandle`) whenever you
need a working else-branch. See
[Concepts](concepts.md#wiring-gates-correctly).

### Unresolved variables pass through as literal text

If `outputVariable` is not set on a producer, or a path is misspelled, the
literal `{{name.field}}` string is substituted and the workflow keeps
running. Read the execution log after the first run and look for `{{` in the
output.

`varCondition` is the exception — it refuses to evaluate a non-numeric
operand and takes neither branch, so a typo cannot route a trade.

### Index symbols need index exchanges

`NIFTY` on `NSE` returns nothing. Use `NSE_INDEX` (and `BSE_INDEX` for
SENSEX/BANKEX). Their options trade on `NFO`/`BFO`.

### Offsets count bars, not days

`offsetBars: 5` on a daily chart is five *trading* bars — seven calendar days
if a weekend intervenes, more across a holiday. `offsetBars: 0` is the last
*closed* bar; today's forming candle is excluded.

### Not every broker supports every interval

`interval` is free text because support varies. Use the `intervals` node to
discover what your broker offers. If an interval is unsupported, the node now
reports the broker's own message.

For unsupported timeframes, download 1-minute data into Historify and set
`source: "db"` — it resamples any minute/hour interval from 1m, and W/M/Q/Y
from D, independent of broker capability.

### History is capped at 200 bars

A 10-year 1-minute request is refused by design (~900,000 rows). The cap
applies to the request window, not just the response, so the download never
happens. Raise `FLOW_MAX_HISTORY_BARS` if you genuinely need more.

### Broker rate limits are tight

Dhan allows 1 request/second on quotes (error 805 on breach); some Zerodha
paths allow 1/second. OpenAlgo also serializes broker history calls behind a
~350 ms gate.

Identical history requests within a run are cached and collapse to one broker
call, but *distinct* requests (different symbol, or the same symbol at a
different interval) each cost a call. A 1-minute schedule with many
indicators across many intervals will feel slow and can trip broker limits.

Prefer one indicator node per distinct symbol/interval and reuse its
`outputVariable`.

### One run at a time

A second trigger arriving while a workflow is still running returns
`already_running` and is dropped — it is not queued. Keep the schedule
interval comfortably longer than the run takes.

### Analyzer mode is global

There is no per-workflow paper-trading switch. Flow inherits the global
Analyzer toggle. Verify which mode you are in before activating a workflow —
the badge is in the header, and order results carry `mode: "analyze"` or
`mode: "live"`.

---

## Pre-flight checklist

Before activating a workflow against live money:

- [ ] Ran it at least once in **Analyzer mode** and read every log line
- [ ] No `{{...}}` placeholders appear unresolved in the log
- [ ] Order quantity units are right (lots vs shares)
- [ ] Every gate evaluated exactly once, with the expected inputs
- [ ] An entry guard exists (`positionCheck` / `fundCheck`) so a repeating
      schedule cannot stack positions
- [ ] The schedule interval is longer than the observed run duration
- [ ] Index symbols use `NSE_INDEX` / `BSE_INDEX`
- [ ] Square-off or exit logic exists, or you are managing exits manually
