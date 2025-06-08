# Why to Build with OpenAlgo?

_“Why should I use OpenAlgo when I can just build my strategy directly on top of the broker’s SDK or API?”_

If you’ve been asking this, you're not alone. Many developers and traders start with direct broker APIs, thinking it’s the fastest way to automate trades. And sure—it works. But then the real issues show up:

How do you monitor trades live?\
Where do you store logs?\
How do you test webhooks before going live?\
How do you fetch and store historical candles?\
What if you want to migrate your strategy from Broker A to Broker B?

That’s when most realize the broker SDK is just the starting point—not the solution.

**OpenAlgo is built to handle all that overhead for you.** It’s a full-stack automation framework, not just an API wrapper. And it’s fully **open source under the AGPL license**, meaning you have the freedom to modify, deploy, and self-host without vendor lock-in or commercial restriction.

<figure><img src=".gitbook/assets/image (110).png" alt=""><figcaption></figcaption></figure>

***

### What Makes OpenAlgo Different?

#### Strategy Management Engine

Define, activate, pause, and monitor strategies—whether they come from Python, TradingView, Amibroker, Excel, N8N, or Chartink. All strategies run through a centralized interface, complete with logs, execution metrics, and security enforcement.

#### Real-Time Quote Layer (Common WebSockets)

Instead of rewriting code for each broker’s WebSocket implementation, OpenAlgo offers a **Common WebSocket layer**. One interface, multiple brokers—streaming normalized market data, ready for use by your strategies.

#### Unified Symbol System

OpenAlgo maintains a **Common Symbol Format**, so a strategy written using one broker’s symbol conventions will continue to work with others—no tedious renaming or mapping required. Whether it's NSE, BSE, or MCX, symbol translation is handled internally.

#### Broker-Agnostic API Layer

All REST APIs for placing orders, fetching positions, or getting quotes are **broker-agnostic**. That means the same API call works whether you're using Zerodha, AngelOne, Dhan, Upstox, or others. Build once, deploy across brokers with minimal change.

#### MCP – Model Context Protocol

OpenAlgo implements **MCP (Model Context Protocol)**, which standardizes the way trade signals are formatted and interpreted. It ensures that whether your signal comes from a Python bot or a TradingView alert, the structure remains consistent—enabling reliable routing, execution, and logging across sources and brokers.

***

### Speed, Stability, and Control

* **HTTP Connection Pooling** is baked into OpenAlgo's broker interfaces. This drops your order latency to **50ms–120ms**, compared to **150ms–250ms** if you hit broker APIs directly via standalone scripts.
* **Historify** module manages all your historical data download needs: 1-minute bars, EOD data, and master contract management are scheduled and stored automatically—no scripts required.
* **API Analyzer** acts as a local sandbox to simulate trades and strategy inputs without sending real orders—perfect for debugging signals.

***

### Deployment Comes Secure by Default

OpenAlgo ships with strong default protections:

* Cross-Origin Resource Sharing (CORS) rules
* Content Security Policy (CSP) headers
* CSRF protection on state-changing routes
* Endpoint rate limiting
* Session management and audit trail logs

You can host it locally, on a VPS, or inside a secured cloud environment without manually configuring each of these layers.

***

### Why Not Just Use Broker APIs Directly?

You certainly can. But here’s what you’d likely need to build on your own:

* Symbol mapping and contract management
* Connection pooling logic
* Live logs and execution dashboards
* Quote stream normalization
* Strategy lifecycle control
* Reusable REST and WebSocket abstractions
* Testing tools for signal simulation
* Historical data download and storage
* Monitoring, recon, and logging infrastructure

OpenAlgo gives you all of this—**pre-wired, tested, documented**, and ready to use.

***

### Open Source That Gives You Real Freedom

OpenAlgo is licensed under **AGPL**, which means:

* You can self-host and modify everything
* You can build commercial systems on top of it (with license compliance)
* No per-order fees, no monthly licenses, no black-box lock-in
* Full access to the source code, strategy engine, and integration layers

You’re not just running a script—you’re running an infrastructure designed to scale with your trading ideas.

***

### Final Thoughts

If you're building your own trading system, don't just look at what a broker's API _can_ do. Look at what it _doesn’t_ do.

OpenAlgo doesn’t replace your strategy logic—it **amplifies it** with the tools needed to operate, monitor, test, and deploy with confidence.

And when the day comes that you want to switch brokers or scale across multiple ones, you’ll be glad your system was built on **OpenAlgo’s broker-agnostic foundation**.

***

