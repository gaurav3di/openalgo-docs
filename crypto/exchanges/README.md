---
description: Delta Exchange
---

# Exchanges

OpenAlgo ships one crypto integration: [Delta Exchange](delta-exchange.md), the `deltaexchange` plugin, covering crypto derivatives (perpetuals, futures and options) plus spot.

It is the only plugin OpenAlgo classifies as `crypto` rather than `IN_stock`, and it is the only one whose instruments sit on the `CRYPTO` exchange. That classification changes real behaviour: spot orders accept fractional quantities, positions carry a configurable leverage, and the daily session expiry is switched off because the market never closes. The [Delta Exchange page](delta-exchange.md) covers API key creation and the settings that differ from a securities broker.

Every other broker in OpenAlgo trades Indian securities and is documented under [Connect Brokers](../../connect-brokers/brokers/README.md).

