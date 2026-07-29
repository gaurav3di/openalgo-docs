# Indicators

The `indicator` node runs any of **118 technical indicators** from the
`openalgo.ta` library over a symbol's history, server-side.

## Basic use

```json
{ "id": "r", "type": "indicator", "position": { "x": 0, "y": 100 },
  "data": {
    "symbol": "RELIANCE", "exchange": "NSE",
    "interval": "D", "source": "api",
    "indicatorName": "rsi",
    "params": "{\"period\": 14}",
    "lookbackBars": 100,
    "tailBars": 5,
    "offsetBars": 0,
    "outputVariable": "rsi"
  } }
```

| Field | Meaning |
| --- | --- |
| `indicatorName` | Function name, lowercase (`rsi`, `sma`, `supertrend`, `macd`, …) |
| `params` | JSON object of the indicator's own arguments, as a string |
| `lookbackBars` | How much history to compute over (capped at 200) |
| `tailBars` | Length of the returned `series` array |
| `offsetBars` | Which bar `at_offset` should read (0 = latest closed) |
| `sourceSeries` | Optional — compute over another series instead of fetching |
| `sourceField` | Optional — which field to read from each `sourceSeries` row |

## Reading the result

```
{{rsi.latest.value}}      value on the most recent closed bar
{{rsi.previous.value}}    value one bar earlier
{{rsi.at_offset.value}}   value offsetBars back
{{rsi.series}}            array of the last tailBars values
{{rsi.series[0].value}}   oldest value in that array
{{rsi.outputs}}           list of output names
{{rsi.bars_used}}         how many bars were actually computed over
```

**Single-output** indicators (`rsi`, `sma`, `ema`, `atr`, …) expose `value`.

**Multi-output** indicators expose `out0`, `out1`, `out2`, … in the library's
own order:

| Indicator | `out0` | `out1` | `out2` |
| --- | --- | --- | --- |
| `macd` | MACD line | signal | histogram |
| `bbands` | upper | middle | lower |
| `supertrend` | level | direction (`-1` up, `1` down) | — |
| `stochastic` | %K | %D | — |
| `adx` | +DI | -DI | ADX |
| `donchian` | upper | middle | lower |

So a supertrend value is `{{st.latest.out0}}` and its direction is
`{{st.latest.out1}}`.

## Historical indicator values

Use `offsetBars` to read a specific bar:

```json
{ "data": { "indicatorName": "supertrend",
            "params": "{\"period\": 10, \"multiplier\": 3}",
            "offsetBars": 5, "outputVariable": "st" } }
```

`{{st.at_offset.out0}}` is the supertrend level five closed bars ago.

You *can* reverse-index `series` instead, but prefer `at_offset`:
`series[0]`'s meaning depends on `tailBars`, so raising `tailBars` from 6 to
10 silently changes which bar `series[0]` refers to. `at_offset` is
unambiguous.

## Nesting indicators

Set `sourceSeries` to another indicator's series to compute an indicator *of*
an indicator — for example a 9-period SMA of RSI:

```json
{ "id": "smoothed", "type": "indicator", "position": { "x": 300, "y": 100 },
  "data": { "indicatorName": "sma", "params": "{\"period\": 9}",
            "sourceSeries": "{{rsi.series}}", "tailBars": 5,
            "outputVariable": "rsiSma" } }
```

Give the upstream node a large enough `tailBars` to feed the nested window —
a 9-period SMA needs at least 9 upstream values.

Only **single-series** indicators can be nested (`sma`, `ema`, `rsi`, `wma`,
`stdev`, `highest`, `lowest`, …). Anything needing independent high/low/close
inputs (`atr`, `supertrend`, `adx`, …) cannot be reconstructed from one
collapsed output series and returns a clear error.

`sourceSeries` also accepts a raw `history` array — `{{h.data}}` uses each
row's `close`. Override with `sourceField` to use `high`, `low`, or a
specific `out1`.

## Crossovers

`crossover`, `crossunder`, and `cross` are **not** available as an
`indicator` node: they need two independent series, and this node reads one
symbol. Build a crossover from two indicator nodes plus an `andGate` — see
[Tutorial 3](tutorials.md#3-crossovers).

`correlation` and `beta` are excluded for the same reason (they compare two
symbols).

## Available indicators

| Category | Functions |
| --- | --- |
| **Trend** | sma, ema, wma, dema, tema, hma, vwma, alma, kama, zlema, t3, frama, trima, mcginley, vidya, alligator, ma_envelopes, supertrend, ichimoku, ckstop |
| **Momentum** | rsi, macd, stochastic, stochf, stochrsi, cci, williams_r, bop, elderray, fisher, crsi, cmo, trix, mom, apo, ppo, po, dpo |
| **Volatility** | atr, natr, true_range, bbands, bbpercent, bbwidth, keltner, donchian, chaikin, rvi, ultimate_oscillator, uo_oscillator, massindex, chandelier_exit, hv, ulcerindex, starc |
| **Volume** | obv, obv_smoothed, vwap, mfi, adl, cmf, emv, force_index, nvi, nvi_with_ema, pvi, pvi_with_signal, volosc, vroc, kvo, pvt, rvol |
| **Oscillators** | roc, rocp, rocr, rocr100, awesome_oscillator, accelerator_oscillator, aroon_oscillator, cho, chop, kst, tsi, vi, stc, gator_oscillator, coppock |
| **Statistical** | linreg, linregangle, linregintercept, lrslope, variance, tsf, median, mode |
| **Hybrid** | adx, adxr, dx, dmi, minus_dm, plus_dm, aroon, pivot_points, psar, fractals, rwi |
| **Price transform** | avgprice, medprice, midprice, midpoint, typprice, wclprice |
| **Utility** | highest, lowest, change, stdev, rising, falling |

Parameter names follow the `openalgo.ta` signatures — `{"period": 14}` for
most, `{"fast_period": 12, "slow_period": 26, "signal_period": 9}` for MACD,
`{"period": 10, "multiplier": 3}` for Supertrend. Required parameters get
sensible defaults if you omit them.
