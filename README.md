# What is OpenAlgo?

OpenAlgo is a free, open source, self hosted algorithmic trading platform that bridges trading ideas with execution. Built with Python Flask and a modern React frontend, it provides a unified API layer across 34 broker plugins: 33 securities integrations and Delta Exchange for crypto derivatives. It supports automation from Amibroker, TradingView, GoCharting, N8N, Python, Java, Go, .NET, Node.js, ChartInk, MetaTrader, Excel, and Google Sheets.

### Overview

**OpenAlgo** is an **open-source, self-hosted algorithmic trading platform** that makes it easy to build, test, and run trading strategies across multiple Indian brokers using a **single, consistent interface**.

Beyond execution, OpenAlgo empowers traders to completely own their trading infrastructure. Traders can effortlessly build algorithmic strategies, indicators, and custom trading dashboards using AI agentic coding tools, connect with their favorite trading platforms, and deploy strategies without being tied to any single broker or vendor. By standardizing broker APIs into one consistent trading layer, OpenAlgo allows strategies to work the same way across brokers, making automation faster, scalable, and fully under the trader’s control.

{% embed url="https://www.youtube.com/watch?v=S5myMo9WUdQ" %}

***

### What Problem Does OpenAlgo Solve?

OpenAlgo is built around one core idea: effortlessly create trading strategies using AI tools and run and monitor them like never before. Instead of learning multiple broker APIs and documentation, AI agentic coding tools such as Codex, Claude Code, Antigravity, and OpenCode understand OpenAlgo directly. Traders can build strategies faster, create their own algo features, design custom trading alerts, connect to any trading platform of their choice, store historical data, and run analytics or backtesting within a fully self hosted environment. There are no code tool offerings available, and you completely own your data and infrastructure.

Most brokers expose their own APIs with different authentication methods, order parameters, market data structures, and operational limitations. OpenAlgo reduces this friction with common request and response shapes. Exchange coverage, authentication, market-data entitlement, GTT support, and other optional capabilities still vary by plugin and broker account.

{% embed url="https://www.youtube.com/watch?v=ImQS-tz_GIo" %}

***

### Who Should Use OpenAlgo?

#### Traders

OpenAlgo is suitable for traders who want to:

* Automate rule-based or discretionary strategies
* Execute TradingView, ChartInk, or Amibroker alerts
* Test strategies safely using sandbox testing
* Monitor orders, positions, and profit/loss in real time
* Maintain full control over execution and data

No coding expertise is required to get started, though advanced users can extend it further.

***

#### Developers

OpenAlgo is designed for developers who need:

* A stable and well-documented REST and WebSocket API
* Uniform request and response formats across brokers
* SDKs in multiple programming languages
* A self-hosted, extensible trading backend
* Support for advanced use cases such as AI-driven execution

{% embed url="https://www.youtube.com/watch?v=es6UUTWtG8Q" %}

***

### Key Capabilities

#### Unified Trading API

OpenAlgo exposes a standardized API that allows you to:

* Place, modify, and cancel orders
* Fetch positions, holdings, and funds
* Access orderbook and tradebook
* Retrieve real-time and historical market data

Broker plugins follow the common API structure for the operations they support.

***

#### Multiple Ways to Run Strategies

OpenAlgo supports different strategy workflows:

* **Hosted Python strategies** running directly inside OpenAlgo
* **Visual Flow strategies** built using a no-code, drag-and-drop interface
* **External signals** from TradingView, Amibroker, ChartInk, Excel, or Google Sheets
* **AI agents** connected through the MCP server

This flexibility allows users to choose the approach that best fits their experience level.

***

#### Sandbox Testing Environment

OpenAlgo includes a fully isolated **sandbox testing environment** for validating strategies before live deployment.

Sandbox testing provides:

* Live market data
* Configurable **sandbox capital**
* Configurable sandbox margin and order behavior
* Configurable MIS square-off schedules
* Simulated order and account state isolated from live broker execution

***

#### Order Control and Safety

OpenAlgo offers optional order approval workflows:

* Automatic execution for fully automated strategies
* Manual approval for supervised or discretionary trading
* Full audit trail of all actions

This helps reduce execution risk and improves transparency.

***

#### Security and Privacy

Security and data ownership are core principles of OpenAlgo:

* Broker session tokens are encrypted; `.env` credentials must be protected by the operator
* Passwords are securely hashed
* Two-factor authentication is supported
* API access is rate-limited
* No user data is collected or shared

All data remains on infrastructure controlled by the user.

***

### Deployment and Usage

OpenAlgo is designed to be self-hosted:

* Run on a local machine
* Deploy on a VPS or cloud server
* Suitable for personal, professional, and team use

Once installed, it can serve as a central execution engine for all trading strategies.

### What OpenAlgo is NOT

Let's be clear about what OpenAlgo doesn't do:

| Misconception         | Reality                                              |
| --------------------- | ---------------------------------------------------- |
| Get-rich-quick scheme | It's a tool - profitability depends on your strategy |
| Strategy provider     | You need your own trading ideas                      |
| Financial advisor     | You're responsible for trading decisions             |
| Black box             | 100% open source - verify every line of code         |
| Cloud service         | Self-hosted - you control everything                 |

### System Requirements

| Component   | Minimum                               | Recommended                  |
| ----------- | ------------------------------------- | ---------------------------- |
| **OS**      | Windows 10, macOS 10.15, Ubuntu 20.04 | Latest versions              |
| **Python**  | 3.12+                                 | 3.12+                        |
| **RAM**     | 2 GB                                  | 2 GB+                        |
| **Storage** | 2 GB                                  | 10 GB+ (for historical data) |
| **Network** | Stable internet                       | Low latency connection       |



### Summary

| Aspect                | OpenAlgo                                         |
| --------------------- | ------------------------------------------------ |
| **Cost**              | Free (Open Source, AGPL License)                 |
| **Brokers**           | 34 plugins: 33 securities integrations + Delta Exchange |
| **Exchanges**         | NSE, NFO, BSE, BFO, MCX, CDS, BCD                |
| **Signal Sources**    | TradingView, Amibroker, ChartInk, Python, AI     |
| **Strategy Building** | Flow (Visual), Python Hosting, External Webhooks |
| **Sandbox Trading**   | Analyzer Mode with ₹1 Crore sandbox capital      |
| **Historical Data**   | Historify with DuckDB storage                    |
| **Real-Time Data**    | WebSocket streaming for quotes and orders        |
| **Notifications**     | Telegram bot, WebSocket updates                  |
| **Data Privacy**      | 100% - self-hosted on your infrastructure        |
| **Skill Required**    | Basic trading knowledge                          |

***
