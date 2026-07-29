# Flow Editor

Flow is OpenAlgo's node-graph strategy builder at `/flow`. You wire market
data into indicators, indicators into conditions, and conditions into order
execution — no Python required.

This section is the **technical reference**. It assumes you are building a
real strategy and want to know exactly what each node does, what the
execution model guarantees, and where the boundaries are.

* [Concepts and Execution Model](concepts.md) — nodes, edges, variables, how a run actually proceeds
* [Node Reference](node-reference.md) — all 60 nodes, their fields and their outputs
* [Market Data and Timeframes](market-data.md) — quotes, history, lookback, bar limits
* [Indicators](indicators.md) — all 118 indicators, lookback, nesting
* [Tutorials](tutorials.md) — eight complete, tested strategies
* [Limitations and Gotchas](limitations.md) — read this before going live

---

## What Flow can do

| Capability | Nodes |
| --- | --- |
| Trigger on a schedule, a webhook, a price level, or an order fill | `start`, `webhookTrigger`, `priceAlert`, `orderUpdateTrigger` |
| Read live quotes and market depth | `getQuote`, `multiQuotes`, `getDepth`, `subscribeLtp/Quote/Depth` |
| Read historical OHLCV at any supported timeframe | `history`, `barOffset`, `priorPeriodOhlc` |
| Compute any of 118 technical indicators | `indicator` |
| Branch on price, time, position, funds, or any computed value | `priceCondition`, `timeWindow`, `timeCondition`, `positionCheck`, `fundCheck`, `varCondition` |
| Combine conditions | `andGate`, `orGate`, `notGate` |
| Place equity, futures, and options orders | `placeOrder`, `smartOrder`, `optionsOrder`, `optionsMultiOrder`, `basketOrder`, `splitOrder` |
| Manage open orders and positions | `modifyOrder`, `cancelOrder`, `cancelAllOrders`, `closePositions` |
| Resolve option symbols, expiries, and chains | `optionSymbol`, `expiry`, `optionChain`, `syntheticFuture` |
| Alert and log | `telegramAlert`, `whatsappAlert`, `log`, `httpRequest` |
| Track P&L for one strategy, not the account | `strategyPnl` |
| Read the account: orders, trades, positions, holdings, funds, margin | `orderBook`, `tradeBook`, `positionBook`, `holdings`, `funds`, `margin`, `getOrderStatus` |
| Check exchange holidays and market timings | `holidays`, `timings` |
| Pause within a run | `delay`, `waitUntil` |
| Arithmetic and state within a run | `mathExpression`, `variable` |

60 node types in total — every one documented in the
[Node Reference](node-reference.md). Every order node calls the same service functions as
`/api/v1/`, so Analyzer (sandbox) mode, Action Center approval, and Telegram
or WhatsApp alerts all behave identically to an API-placed order.

## What Flow cannot do

These are hard boundaries, not oversights. [Limitations](limitations.md)
covers each in detail with workarounds.

| Limitation | Practical effect |
| --- | --- |
| **No state across runs** | `variable` values reset every run. "Have I already entered today?" must be answered from the broker (`positionCheck`, `orderBook`), not a counter. |
| **`delay` and `waitUntil` block the run** | They hold the execution slot for their full duration. Waiting out a session means a second workflow on its own schedule, not a six-hour `waitUntil`. |
| **No loops** | You cannot iterate a symbol list. One workflow handles one symbol; clone it per symbol. |
| **No backtesting** | Flow runs forward only. Use the Python Strategy Host or the `openalgo` SDK to backtest. |
| **No pandas objects** | Variables are JSON. You get arrays of records and single values, not a `pandas.Series`. |
| **Max 200 bars per fetch** | Deep history (10 years of 1-minute data) is refused by design. See [Market Data](market-data.md). |
| **Triggers cannot read upstream data** | A trigger is the entry point, so its fields cannot contain `{{variables}}`. |
| **Indicators need one symbol** | `correlation`, `beta`, `crossover`, `crossunder` need two independent series and are not available as a single `indicator` node. Crossovers are built from two indicator nodes — see [Tutorial 3](tutorials.md#3-crossovers). |

## Live vs Analyzer mode

Flow does not have its own paper-trading switch. It inherits the global
**Analyzer** toggle:

* **Analyzer on** — orders are simulated by the Sandbox engine. Order ids are
  real-looking, `mode` is `analyze`, and nothing reaches your broker.
* **Analyzer off** — orders go to the live broker.

Always build and validate a strategy with Analyzer on. Every example in this
section was verified in Analyzer mode.
