# Why to Build with OpenAlgo?

_“Why should I use OpenAlgo when I can just build my strategy directly on top of the broker’s SDK or API?”_

It’s a common question. Many start with broker SDKs because it feels quick—just wire your signals and send orders. But soon, the pain points show up:

* How do you monitor trades live?
* Where do you store and replay logs?
* How do you test webhooks or strategies before going live?
* How do you manage symbols, expiries, and contracts?
* What happens when you want to move from Broker A to Broker B?

That’s when you realize the SDK is not enough.

**OpenAlgo takes care of the heavy lifting.**\
It’s not just an API wrapper—it’s a **full-stack, open-source trading automation framework** designed to host strategies, manage brokers, and scale securely.

<figure><img src=".gitbook/assets/image (123).png" alt=""><figcaption></figcaption></figure>

***

#### What Makes OpenAlgo Different?

**Strategy Management & Hosting**

You can now **host your Python strategies directly inside OpenAlgo**, alongside strategies from TradingView, Amibroker, MetaTrader, Excel, N8N, or Chartink. Start, pause, monitor, and analyze—all from a central control plane.

**Backtesting & API Analyzer**

Integrated with **Backtrader**, OpenAlgo lets you backtest signals using historical data before pushing them live. The **API Analyzer** works like a local sandbox—test your signals, APIs, and strategies without hitting real broker servers.

**Cache Layer & Log Storage**

High-performance caching keeps repeated requests fast, while **centralized log storage** ensures every trade, event, and error is traceable for compliance and debugging.

**Multi-Broker, Multi-Platform**

OpenAlgo supports **multiple brokers** via a **common API and WebSocket layer**. Write your strategy once, and run it across Zerodha, AngelOne, Dhan, Upstox, Flattrade, Firstock, and more—without rewriting code.

It also supports running strategies from **TradingView, Amibroker, MetaTrader, Chartink, Excel, Google Sheets, and N8N**—all via the same API.

**Unified Symbol & Contract Management**

With OpenAlgo’s **Common Symbol Format**, you don’t have to worry about broker-specific quirks. Contracts, expiries, and lot sizes are maintained automatically.

***

#### Speed, Stability, and Control

* **HTTPX with Connection Pooling** reduces latency to **50ms–120ms** vs. 150ms–250ms in plain scripts.
* **Backtesting integration** with Backtrader lets you validate performance before risking real money.
* **WebSocket broadcast layer** ensures one stream can power multiple strategies, cutting redundant broker connections.
* **Latency Monitor** tracks order round-trips so you know exactly how your infra is performing.

***

#### Deployment Comes Secure by Default

OpenAlgo is battle-tested with:

* CORS and CSP headers
* CSRF protection
* Endpoint rate limiting
* Session management and audit trails
* Sandboxed execution for hosted strategies

You can run it locally, inside **Docker**, or on cloud servers—secure out of the box.

***

#### SDKs, Add-ins, and Community Support

* **Python, Go, Node.js SDKs** to code in your favorite language
* **Excel Add-ins** for traders who love spreadsheets
* **Docker support** for clean deployment
* **Community support** through Discord and GitHub
* **Log storage** for auditing and debugging

***

#### Why Not Just Use Broker APIs Directly?

You’d have to reinvent:

* Strategy hosting and lifecycle control
* Backtesting and simulation tools
* Symbol and contract management
* Connection pooling and WebSocket normalization
* Trade management dashboards
* Cache and log storage layers
* Monitoring and latency measurement

OpenAlgo ships with all this—**pre-wired, tested, and open source**.

***

#### Open Source Freedom

Licensed under **AGPL**, OpenAlgo gives you:

* Full source code and modification rights
* Freedom to self-host
* No per-order fees or vendor lock-ins
* The ability to build commercial apps on top (with compliance)

***

#### Final Thoughts

Broker APIs give you _access_.\
OpenAlgo gives you _infrastructure_.

It doesn’t replace your strategy logic—it **amplifies** it with the ecosystem you need to operate, monitor, backtest, and scale confidently.

And when you’re ready to switch brokers or expand to multi-broker setups, you’ll already be on **OpenAlgo’s unified, broker-agnostic foundation**.

***

