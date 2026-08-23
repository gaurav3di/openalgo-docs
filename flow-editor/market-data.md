# Market Data and Timeframes

## Live quotes

`getQuote` returns a single symbol's snapshot:

```json
{ "id": "q", "type": "getQuote", "position": { "x": 0, "y": 100 },
  "data": { "symbol": "RELIANCE", "exchange": "NSE", "outputVariable": "q" } }
```

Available fields: `{{q.data.ltp}}`, `.open`, `.high`, `.low`, `.prev_close`,
`.volume`, `.oi`, `.bid`, `.ask`.

`.high` and `.low` are **today's running session high and low**, useful for
stateless "did price come back to this level today?" logic without needing
cross-run memory. [Tutorial 5](tutorials.md#5-previous-day-breakout-with-a-gap-filter)
uses exactly this.

Use `multiQuotes` for several symbols in one call, and `getDepth` for the
order book.

## Historical bars

Three nodes read history, each answering a different question.

| Node | Question it answers |
| --- | --- |
| `history` | "Give me the raw OHLCV array for a date range." |
| `barOffset` | "What was the close N bars back?" |
| `priorPeriodOhlc` | "What was the previous day/hour/week/month's OHLC?" |

### barOffset: N bars back

```json
{ "id": "b5", "type": "barOffset", "position": { "x": 0, "y": 100 },
  "data": { "symbol": "NIFTY", "exchange": "NSE_INDEX", "interval": "D",
            "source": "api", "offsetBars": 5, "outputVariable": "bar5" } }
```

Exposes `{{bar5.open}}`, `.high`, `.low`, `.close`, `.volume`, `.timestamp`.

**Offsets count bars, not calendar days.** `offsetBars: 0` is the most recent
*closed* bar, today's still-forming candle is excluded. On a Wednesday,
`offsetBars: 5` on a daily chart lands on the previous Tuesday: five trading
bars back, which spans seven calendar days across the weekend.

Because it counts bars, the same node covers "5 hours back" (`interval: "1h"`)
or "30 minutes back" (`interval: "1m", offsetBars: 30`).

### priorPeriodOhlc: previous period levels

```json
{ "id": "pd", "type": "priorPeriodOhlc", "position": { "x": 0, "y": 100 },
  "data": { "symbol": "NIFTY", "exchange": "NSE_INDEX",
            "period": "previous_day", "source": "api", "outputVariable": "pd" } }
```

`period` accepts `previous_hour`, `previous_day`, `previous_week`,
`previous_month`. Weekly and monthly are aggregated from daily bars, high is
the max, low the min, open the first, close the last of the completed period.

Convenience aliases are provided alongside the raw fields:

| Alias | Same as |
| --- | --- |
| `{{pd.pdh}}` | `{{pd.high}}` |
| `{{pd.pdl}}` | `{{pd.low}}` |
| `{{pd.pdc}}` | `{{pd.close}}` |

The node never returns a still-forming candle. If history is too short to
contain a completed prior period, it raises rather than silently handing back
today's partial bar.

## Timeframes

`interval` is a **free-text field, not a dropdown**, because broker support
varies. Common values: `1m`, `3m`, `5m`, `15m`, `30m`, `1h`, `D`.

Use the `intervals` node to list what your connected broker actually
supports:

```json
{ "id": "iv", "type": "intervals", "position": { "x": 0, "y": 100 },
  "data": { "outputVariable": "ivs" } }
```

If you request an interval the broker does not offer, the node now reports
the broker's own message and points you at the alternative rather than
failing with a misleading "no data".

### Resampling with the Historify DB

Set `source: "db"` to read from OpenAlgo's local Historify store instead of
the broker. Historify stores `1m` and `D` and **computes everything else on
demand with SQL**:

| From stored | You can request |
| --- | --- |
| `1m` | any minute or hour interval, `2m`, `3m`, `4m`, `25m`, `2h`, … |
| `D` | `W`, `M`, `Q`, `Y` |

This is the answer to "my broker does not support 3-minute candles". Download
1-minute data once into Historify, then every node can request `3m`
regardless of broker capability.

The symbol must already be in Historify, download it from the Historify page
first.

## The 200-bar ceiling

**Every history fetch is capped at the most recent 200 bars.**

This is deliberate. Ten years of daily data is ~2,500 rows, but ten years of
1-minute data is ~900,000 rows, a download that takes minutes, exhausts the
broker's rate budget, and then sits in workflow memory. No indicator here
needs that depth; a 200-period moving average is the deepest common window.

The cap is applied when **sizing the request**, not merely when trimming the
response, so the oversized fetch never leaves OpenAlgo:

| Interval | Calendar window actually requested |
| --- | --- |
| `1m` | ~5 days |
| `5m` | ~9 days |
| `15m` | ~17 days |
| `1h` | ~50 days |
| `D` | ~325 days |
| `W` | ~4.4 years |
| `M` / `Q` / `Y` | capped at ~11 years |

A second ceiling bounds the calendar span, because 200 quarterly bars would
otherwise span 54 years and 200 yearly bars 219 years, ranges no broker
serves sensibly.

The `history` node's explicit `startDate`/`endDate` is narrowed the same way.
Requesting `2016-01-01` to `2026-07-29` on a daily chart silently becomes
roughly the last 200 daily bars, and the execution log records it.

Both limits are tunable if a strategy genuinely needs more depth:

```
FLOW_MAX_HISTORY_BARS=200
FLOW_MAX_HISTORY_CALENDAR_DAYS=4000
```

## Rate limits and caching

Broker data APIs are throttled far more tightly than a node graph suggests:

| Broker | Limit |
| --- | --- |
| Dhan | 5 req/s history, **1 req/s quotes** (error 805 on breach) |
| Flattrade | 10 req/s |
| Zerodha | 1 req/s on some paths |

On top of that, OpenAlgo serializes every broker history call behind a
process-wide ~350 ms gate.

A strategy asking for RSI + SMA + ATR + previous-day levels on one symbol
would issue four *identical* history requests per run. To prevent that, all
history-reading nodes share a short-TTL cache keyed by the exact request
(symbol, exchange, interval, dates, source). Four nodes wanting the same
series produce **one** broker call.

Errors and empty responses are never cached, so a transient failure stays
retryable. Tune with:

```
FLOW_HISTORY_CACHE_TTL=30        # seconds; 0 disables
FLOW_HISTORY_CACHE_MAXSIZE=256
```

Practical guidance: prefer one `indicator` node per distinct
symbol/interval/indicator, and reuse its `outputVariable` downstream rather
than adding duplicate nodes.

## Index symbols

Index underlyings use the index exchanges, not the equity ones:

| Symbol | Exchange |
| --- | --- |
| `NIFTY`, `BANKNIFTY`, `FINNIFTY`, `MIDCPNIFTY` | `NSE_INDEX` |
| `SENSEX`, `BANKEX`, `SENSEX50` | `BSE_INDEX` |
| `RELIANCE`, `SBIN`, … | `NSE` / `BSE` |

Their options trade on `NFO` (NSE) and `BFO` (BSE). Using `NSE` for `NIFTY`
returns no data.
