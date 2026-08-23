# Concepts and Execution Model

Understanding how a run actually proceeds is the difference between a
strategy that fires once and one that fires twice.

## Nodes, edges, handles

A workflow is a set of **nodes** joined by **edges**. Every node has an `id`,
a `type`, a canvas `position`, and a `data` object holding its configuration.

```json
{ "id": "n2", "type": "getQuote", "position": { "x": 0, "y": 100 },
  "data": { "symbol": "RELIANCE", "exchange": "NSE", "outputVariable": "q" } }
```

Edges connect a `source` node to a `target` node:

```json
{ "id": "e1", "source": "n1", "target": "n2" }
```

Condition nodes have **two output handles**. An edge leaving one carries a
`sourceHandle`:

```json
{ "id": "e2", "source": "cond", "sourceHandle": "true",  "target": "buy" }
{ "id": "e3", "source": "cond", "sourceHandle": "false", "target": "skip" }
```

An edge from a condition **without** a `sourceHandle` is a *pass-through*:
it is followed no matter which way the condition went. This is how you attach
logging or alerting that should see every evaluation.

> Handle vocabulary differs by node: `positionCheck`, `fundCheck`,
> `priceCondition`, `timeWindow`, and `varCondition` use `"true"`/`"false"`,
> while `timeCondition` and `notGate` use `"yes"`/`"no"`. The executor treats
> them as synonyms, but match the node's own vocabulary so saved graphs read
> cleanly.

## Variables and interpolation

Most data nodes accept an `outputVariable`. Set it, and the node's whole
result is stored under that name for every downstream node to read with
`{{...}}`.

```json
{ "type": "getQuote", "data": { "symbol": "RELIANCE", "exchange": "NSE", "outputVariable": "q" } }
```

Downstream: `{{q.data.ltp}}`, `{{q.data.open}}`, `{{q.data.prev_close}}`.

Path syntax:

| Form | Example |
| --- | --- |
| Nested keys | `{{order.data.orderid}}` |
| Array index (positive only) | `{{expiries.data[0]}}` |
| Combined | `{{chain.data.results[0].ce.ltp}}` |

Built-ins are always available:

| Variable | Example | Notes |
| --- | --- | --- |
| `{{date}}` | `2026-07-30` | Server calendar date |
| `{{session_date}}` | `2026-07-30` | Trading session date. Differs from `{{date}}` between midnight and the 03:00 IST rollover. |
| `{{time}}`, `{{timestamp}}`, `{{iso_timestamp}}` | | |
| `{{hour}}`, `{{minute}}`, `{{second}}` | | |
| `{{day}}`, `{{month}}`, `{{year}}` | | |
| `{{weekday}}` | `Thursday` | A name, so it cannot be compared numerically |
| `{{weekday_num}}` | `4` | 1 = Monday. Use this in a condition. |
| `{{quarter}}` | `3` | 1 through 4 |
| `{{week_of_year}}`, `{{day_of_year}}` | `31`, `211` | ISO week, day of year |

For "has a new day, week, month or quarter started", use the
[`calendar` node](node-reference.md) rather than comparing these, it accounts
for weekends and exchange holidays, which a `{{day}} == 1` test does not.

**If a path does not resolve, the literal `{{...}}` text is passed through.**
The workflow does not stop. This is deliberate, it makes typos visible in
the execution log rather than crashing a live strategy, but it means you
should read your logs after the first run. A `varCondition` given an
unresolved operand is the one exception: it refuses to evaluate rather than
treating the text as zero.

## Execution order

A run starts at the single trigger node and walks the graph **depth-first**,
following each outgoing edge in order. There is no parallelism and no
scheduler, one node finishes completely before the next begins.

Two consequences matter:

**1. Order of nodes is the order you wire them.** If a `varCondition` reads
`{{rsi.latest.value}}`, the `indicator` node producing `rsi` must be an
*ancestor*, not a sibling.

**2. Logic gates wait, then fire once.** An `andGate` with two inputs is
reached first by whichever input finishes first. The gate detects that its
other input has not been evaluated yet and skips, leaving a log line:

```
andGate: waiting for 1 more input(s) before evaluating
```

When the second input completes, its traversal reaches the gate again, now
with a full set, and the gate evaluates exactly once:

```
AND Gate: [True, True] -> True
```

## Wiring gates correctly

This is the single most common mistake. There are two ways to feed a gate,
and they behave differently.

**Pass-through wiring (recommended).** Edges carry only `targetHandle`:

```json
{ "id": "e3", "source": "c1", "target": "gate", "targetHandle": "input-0" }
{ "id": "e4", "source": "c2", "target": "gate", "targetHandle": "input-1" }
```

Both conditions always reach the gate, so the gate sees `[True, False]` and
can evaluate to false, meaning **its `false` branch works**.

**True-handle wiring.** Edges carry `sourceHandle: "true"`:

```json
{ "id": "e3", "source": "c1", "sourceHandle": "true", "target": "gate", "targetHandle": "input-0" }
```

A false condition never follows its true edge, so the gate is never reached
at all. The gate can only ever produce `true`, and **its `false` branch is
unreachable**. That is fine when you only care about the entry path, but do
not attach an else-branch to it.

Always pin gate inputs with `targetHandle: "input-0"`, `"input-1"`, … up to
`inputCount - 1`.

## Triggers

A workflow must contain exactly one trigger, and it must be one of:

| Trigger | Fires when |
| --- | --- |
| `start` | A schedule elapses (`once`, `daily`, `weekly`, `interval`) |
| `webhookTrigger` | An external system POSTs to the workflow's webhook URL |
| `priceAlert` | A polled LTP condition is met (one-shot by default) |
| `orderUpdateTrigger` | A matching order changes status (fill, reject, cancel) |

A second trigger does not raise an error, it is **silently ignored**. The
executor walks the graph from the first trigger it finds, so the second
trigger and everything downstream of it simply never runs. Verified: a
workflow with two `start` nodes feeding two separate log branches produced
output from one branch only. If part of your graph mysteriously never
executes, count the triggers first.

Because one trigger drives the whole graph, every branch shares the data nodes
above it. That is also the cheapest layout: one `getQuote` feeding six
branches is one broker call, whereas six separate workflows make six. Quotes
and the order book are not cached, so the split layout is what trips a rate
limit first.

Triggers are entry points, so **their configuration cannot reference
`{{variables}}`**, there is no upstream node to resolve against. The
`orderUpdateTrigger` rejects an interpolated Order ID with a clear 400 rather
than storing a placeholder that could never match.

`marketHoursOnly: true` on a `start` node pauses the schedule outside
09:15-15:30 IST on weekdays.

## Execution limits

| Limit | Value |
| --- | --- |
| Max node depth | 100 |
| Max node visits per run | 500 |
| Concurrent runs of one workflow | 1 (a second trigger returns `already_running`) |
| Max bars per history fetch | 200 (see [Market Data](market-data.md)) |


## Editing a workflow as JSON

The editor exports and re-imports JSON, which is how a strategy can be authored
or reviewed as a file, or generated by an AI from the import spec.

**Import always creates a new workflow.** That is deliberate for sharing, but it
means iterating on your own strategy would leave copies behind and change the
webhook URL every time.

To update a workflow you already have, use **Replace from JSON** in the workflow
menu. It swaps the nodes and edges in place and keeps:

- the workflow id and its `/flow/editor/<id>` URL
- the webhook token and secret, so anything posting to it keeps working
- the active state and the stored API key

It is validated exactly like Import: an incomplete graph is rejected with the
reason rather than saved to fail later, and legacy fields from an older export
are upgraded on the way in.

One thing to watch: node edits apply from the next run, because the graph is
re-read every time. The **trigger** is different - its schedule and any price or
order watch are registered when you activate. If you change the trigger, the
editor tells you to deactivate and reactivate.

For scripted use there is also
`scripts/update_flow_workflow.py --id <id> --file strategy.json`, with
`--dry-run` to preview.
