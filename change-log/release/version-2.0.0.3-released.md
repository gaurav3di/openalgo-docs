# Version 2.0.0.3 Released

#### Version 2.0.0.3 Released

**Date: 13th Apr 2026**

**Feature Release: Async-to-Sync WebSocket Migration, Market Price Protection (MPP), Samco 2FA, Shoonya OAuth, Multi-Device Session Tracking & Security Hardening**

This is a feature release featuring **59 commits** (excluding auto-builds) from **7 contributors**, focused on migrating broker WebSocket adapters from asyncio to sync for Docker/eventlet compatibility, Market Price Protection (MPP) for Zerodha/Kotak/Samco/Tradejini, Samco 2FA authentication, Shoonya OAuth migration, multi-device session tracking, Telegram bot enhancements, and security hardening.

***

**Highlights**

* **Async-to-Sync WebSocket Migration** — Converted Zerodha, Upstox, and mstock streaming adapters from asyncio `websockets` to sync `websocket-client`, eliminating greenlet conflicts in Docker (Gunicorn + eventlet) deployments
* **Market Price Protection (MPP)** — Added automatic MARKET/SL-M to LIMIT/SL conversion with MPP for Zerodha, Kotak, Samco, and Tradejini, protecting traders from extreme price slippage
* **Samco 2FA Authentication** — Complete Samco 2FA implementation with OTP, secret key, IP management, and order status normalization
* **Shoonya OAuth Migration** — Migrated Shoonya from TOTP login to OAuth (GenAcsTok) authentication with WebSocket and SL-M MPP fixes
* **Multi-Device Session Tracking** — Session resume support across devices with active session monitoring and login activity tracking
* **Telegram Bot Commands** — New `/closeall` and `/mode` Telegram commands for remote position management and mode switching
* **Dynamic Browser Tab Titles** — All 80+ pages now show descriptive titles in browser tabs (e.g., "OI Tracker | OpenAlgo")
* **Synthetic Future API Optimization** — Reduced broker API calls from 4+ to 2 using multiquotes batching, cutting Dhan response time from \~8s to \~2s
* **Multi-Arch Docker Images** — CI/CD now builds both amd64 and arm64 Docker images on native runners
* **Security Patches** — Updated cryptography, axios, vite, Pygments, and lodash-es for CVE fixes

***

**New Features**

**Async-to-Sync WebSocket Migration**

* Convert Zerodha streaming from async websockets to sync websocket-client
* Convert Upstox streaming from async websockets to sync websocket-client
* Convert mstock streaming from async websockets to sync websocket-client
* Reset Upstox and mstock reconnect counters on successful connection

**Market Price Protection (MPP)**

* Enable market protection for Zerodha market orders with decimal strike preservation
* Add MPP for Kotak market orders (MARKET → LIMIT, SL-M → SL with tick-based price protection)
* Convert MARKET/SL-M orders to LIMIT/SL with MPP for Samco
* Add MPP and fix orderbook status mapping for Tradejini
* Guard `trigger_price` float conversion in SL-M MPP handler

**Samco 2FA Authentication**

* Implement Samco 2FA authentication with OTP, secret key, and IP management
* Refactor Samco 2FA to auth table with `aux_param` columns
* Fix order status normalization for Cancel/Modify buttons in orderbook
* Resolve FD leaks and resource issues in Samco WebSocket streaming
* Use equity segment only for Samco funds to avoid double-counting

**Shoonya OAuth Migration**

* Migrate Shoonya from TOTP login to OAuth (GenAcsTok) authentication
* Fix Shoonya WebSocket, SL-M MPP conversion, and circular import
* Fix review issues: input validation, null safety, log redaction

**Multi-Device Session & Login Activity**

* Multi-device session resume and active session tracking
* Login activity tracking and security dashboard tabs
* Resolve file descriptor and memory leak from uncleaned database sessions

**Telegram Bot Enhancements**

* Add `/closeall` and `/mode` Telegram bot commands for remote trading control
* Fix `/chart` rendering in Docker and eventlet environments

**Frontend Enhancements**

* Dynamic browser tab titles for all 80+ pages (#881)
* Copy and download buttons for Python strategy logs (#1233)
* Add aria-labels to TradeBook buttons for accessibility (#1189)

**Tradejini Broker Improvements**

* Add index symbol normalization and fix WebSocket exchange mapping
* Prevent WebSocket FD leaks in quotes, depth, and streaming
* Add MPP and fix orderbook status mapping

**Smart Order Optimization**

* Remove 500ms smart order delay, add position cache + per-symbol lock for faster execution

**Infrastructure**

* Build multi-arch Docker images (amd64 + arm64) on native runners
* Replace `traceback.print_exc()` with `logger.exception()` across all REST API endpoints (#1012)

***

**Bug Fixes**

* Fix: prevent duplicate order execution race condition in sandbox mode
* Fix: optimize synthetic future API — reduce broker API calls from 4+ to 2 using multiquotes (#855)
* Fix: return mock response object instead of None for MPP ValueError
* Fix: address cubic-dev P1 issues in order\_api and auth\_api
* Fix: IIFL Capital index symbol normalization and quotes update
* Fix: add IIFL Capital to install script files
* Fix: add missing security headers to Flask responses (#1014)
* Fix: standardize error logging with `logger.exception()` (#1017)
* Fix: adjust font sizes and cell heights in seasonality heatmap
* Fix: refine monthly data fetching and processing in seasonality
* Fix: update openalgo SDK to 1.0.47, remove numba/llvmlite dependencies

***

**Security**

* Bump cryptography to 46.0.7 (security patch)
* Bump vite to 7.3.2 (security patch)
* Bump axios to 1.15.0 (security patches)
* Upgrade cryptography, Pygments, and lodash-es for security

***

**Contributors**

Special thanks to all contributors who made this release possible:

* @Kalaiviswa (kalaivani) — Async-to-sync WebSocket migration (Zerodha, Upstox, mstock), Tradejini MPP/FD fixes, Shoonya OAuth migration, Samco 2FA, multi-device session tracking
* @marketcalls — Synthetic future optimization, browser tab titles, strategy log buttons, Telegram commands, smart order optimization, seasonality fixes
* @Na1neeth (Navaneeth) — Samco 2FA authentication and WebSocket FD fixes
* @Chessing234 (Taksh) — Security headers and error logging standardization
* @LuckyAnsari22 — Database docstrings, TradeBook accessibility
* @mvanhorn — Replace traceback.print\_exc() with logger.exception()

***

**Links**

* **Repository**: [https://github.com/marketcalls/openalgo](https://github.com/marketcalls/openalgo)
* **Documentation**: [https://docs.openalgo.in](https://docs.openalgo.in)
* **Discord**: [https://www.openalgo.in/discord](https://www.openalgo.in/discord)
* **YouTube**: [https://www.youtube.com/@openalgo](https://www.youtube.com/@openalgo)

***

### \[2.0.0.0] - 2026-01-22

#### Major Release: Complete Frontend Rewrite & Feature Expansion

This is a major release featuring a complete rewrite of the frontend from Flask/Jinja2 templates to a modern React 19 Single Page Application (SPA). This release includes **212 commits** representing months of development work, introducing new features like Flow Visual Builder, Historify, and enhanced real-time capabilities.

***

### Highlights

* **React 19 Frontend** - Complete migration of 77 templates to modern React with TypeScript
* **Flow Visual Builder** - Node-based visual workflow builder for trading automation
* **Historify** - Historical market data management with DuckDB storage
* **Real-Time WebSocket** - Native WebSocket integration for live market data
* **Sandbox Mode** - Enhanced sandbox testing environment with sandbox capital
* **API Playground** - Bruno-style API testing with WebSocket support
* **Python Strategies** - Enhanced scheduler with real-time status and resource limits
* **Telegram Bot** - Fixed callbacks and improved status display
* **Enhanced Security** - Multiple security improvements and vulnerability fixes

***

### New Features

#### React 19 Frontend Migration (77 Templates)

**Phase 1 - Foundation**

* Initialized React frontend with Vite, TypeScript, TanStack Query
* Added Flask blueprint to serve React frontend
* Pre-built frontend dist included for community use

**Phase 2 - Core Authentication & Trading**

* Login, Dashboard, Profile pages
* Orders, Positions, Holdings pages
* Order placement and management

**Phase 3 - Search & Symbol Management**

* FNO Discovery with performance optimization
* Symbol search and watchlist
* Bulk watchlist operations

**Phase 4 - Charts, WebSocket & Sandbox**

* TradingView charts integration
* WebSocket Test Console
* Sandbox/Analyzer mode interface

**Phase 5 - Platform Integrations**

* TradingView webhook page
* GoCharting integration
* Amibroker integration
* ChartInk integration

**Phase 6 - Strategy & Automation**

* Python Strategies management
* Strategy scheduler with SSE
* Strategy logs viewer

**Phase 7 - Monitoring & Administration**

* Logs, Latency Monitor, Traffic Logs
* Profile & Security settings
* Action Center for order approval
* Admin & Telegram modules

**Frontend Tech Stack**

* React 19 with TypeScript
* Vite 6 build system with code splitting
* TanStack Query v5 for server state
* shadcn/ui + Tailwind CSS 4 + DaisyUI
* Biome.js (replaced ESLint)
* Vitest unit tests + Playwright E2E tests
* Responsive mobile bottom navigation
* Accessibility testing (jest-axe)

***

#### Flow Visual Builder

* **Node-based visual workflow builder** for trading strategies
* **Order Nodes**: Market Order, Limit Order, Smart Order, Basket Order
* **Options Order Node**: ATM/ITM/OTM offset resolution for F\&O
* **Modify Order Node**: Live order management within workflows
* **Cancel Order Node**: Cancel single or all orders
* **Close Position Node**: Square off positions
* **WebSocket Streaming Nodes**: Real-time data within workflows
* **Telegram Alert Node**: Send notifications from workflows
* **Webhook Integration**: Trigger flows from external systems
* **Multi-leg Options Strategy**: Execute complex option strategies
* **Keyboard Shortcuts**: Efficient workflow creation
* Service integration for order execution

***

#### Historify - Historical Data Management

* **DuckDB-powered storage** for historical market data
* **Multi-timeframe support**: 1m, 5m, 15m, 30m, 1h, Daily
* **Computed timeframes**: Weekly (W), Monthly (MO), Quarterly (Q), Yearly (Y)
* **Aggregation from daily data** for higher timeframes
* **Bulk export** with inline symbol selection
* **Multi-timeframe export** in single operation
* **Parquet import support** for external data sources
* **TradingView-style charts** with IST timezone
* **Styled crosshair tooltips** with IST timestamps
* **Job management**: Pause, resume, cancel operations
* **Broker badge display** and theme toggle
* **Date selector improvements** with Calendar component
* **Exchange market open time alignment** for candle boundaries

***

#### Real-Time WebSocket Integration

* **Native WebSocket** for Holdings and Positions pages
* **Unified WebSocket proxy server** on port 8765
* **ZeroMQ message bus** for high-performance data distribution (port 5555)
* **Connection pooling**: MAX\_SYMBOLS\_PER\_WEBSOCKET (1000) x MAX\_WEBSOCKET\_CONNECTIONS (3)
* **MultiQuotes API fallback** when WebSocket unavailable
* **Market timing awareness** for automatic data source switching
* **Real-time P\&L calculation** using live LTP data
* **WebSocket templates** in Playground with Bruno-style collections
* **Multi-client subscribe/unsubscribe** support
* **Callback-based data retrieval** for Flow nodes
* **Pong message display** for manual ping testing

***

#### Sandbox Mode (Sandbox Testing)

* **Isolated sandbox trading** with Rs. 1 Crore sandbox capital
* **Realistic margin system** with leverage
* **Auto square-off** at exchange timings for F\&O contracts
* **Complete isolation** from live trading
* **Separate database** (sandbox.db) for sandbox trades
* **Real-time P\&L** using WebSocket data
* **Session-based position filtering** for expired contracts
* **Expired F\&O contract cleanup** on app startup
* **Sandbox logs** with date filter and Calendar icons
* **Wide dialog display** (98vw) for better visibility

***

#### API Playground

* **Bruno-style API collection browser**
* **WebSocket testing console** with comprehensive controls
* **CodeMirror JSON editor** with syntax highlighting
* **Theme support** matching application theme
* **Manual ping/pong testing** for WebSocket connections
* **Multiple tabs** for endpoints with same path but different names
* **Nested braces handling** in body:json parsing
* **Source parameter** for History API collections

***

#### Python Strategies

* **Enhanced scheduler** with mandatory scheduling
* **Real-time status updates** via SSE (Server-Sent Events)
* **Resource limits** to prevent runaway strategies
* **Python Strategy Guide page** with comprehensive help
* **FAQ for installing libraries** (TA-Lib, pandas-ta, etc.)
* **Log management** with configurable retention
* **Reverse chronological logs** with auto-scroll
* **Schedule box theme** with opacity-based dark mode colors
* **Holiday enforcement** for market-aware scheduling
* **Environment Variables feature removed** (security)

***

#### Telegram Bot

* **Fixed /menu callbacks** for command navigation
* **Fixed /status display** for current position status
* **Flow Telegram alert integration** using existing send\_alert\_sync
* **Admin & Telegram modules** migrated to React

***

#### Email & SMTP

* **Fixed SMTP email delivery**
* **Updated email templates**
* **Email icon centering** using table-based layout

***

#### Action Center

* **Order approval workflow** for managed accounts
* **Semi-Auto mode** for manual approval
* **Auto mode** for direct execution
* **Complete migration** to React interface
* **Documentation** added (Module 42)

***

### Improvements

#### User Interface

* Profile menu with mode controls on all pages
* Theme consistency across broker and public pages
* Theme sensitivity for dark/light mode switching
* Broker badge display across pages
* Chart icons in watchlist for smart navigation
* Responsive dialogs with optimized widths
* Mobile bottom navigation
* Accessible icon buttons with aria-labels

#### Performance

* FNO Discovery performance optimization
* Historify storage optimization
* Code splitting and lazy loading
* Bulk watchlist add optimization
* Connection pooling for WebSocket

#### Order Management

* P\&L % calculation for flat positions using implied investment
* Show dash for P\&L % on closed positions
* Preserved realized P\&L for closed positions
* Position filtering for session boundaries
* Show closed positions that were traded today
* Expired F\&O contract cleanup on startup
* Order field names aligned with OpenAlgo schema

#### Broker Integrations

* AliceBlue holdings symbol field fix
* OAuth broker redirect improvements (AJAX vs browser detection)
* Broker login migrated to React JSON responses
* Updated lot sizes and expiry dates in Bruno collections
* Broker credentials GUI for easy configuration

#### Charts

* TradingView-style x-axis labels for daily+ timeframes
* IST timezone correction for W/MO/Q/Y timeframes
* Dates instead of time for daily+ timeframes
* CodeMirror JSON editor on TradingView and GoCharting pages

***

### Security

* Fixed critical frontend vulnerabilities
* Removed environment variables feature from Python strategies
* Added resource limits for strategy execution
* Enhanced CSRF protection
* Security audit documentation added
* Dependency updates for known vulnerabilities

***

### Documentation

#### User Guide (30 Modules)

* What is OpenAlgo, Key Concepts, System Requirements
* Installation Guide, First-Time Setup
* Broker Connection, Dashboard Overview
* Understanding Interface, API Key Management
* Order Types, Smart Orders, Basket Orders
* Positions & Holdings, Analyzer Mode
* Symbol Format Guide
* TradingView, Amibroker, ChartInk, GoCharting Integration
* Python Strategies, Flow Visual Builder
* Action Center, Telegram Bot
* PnL Tracker, Latency Monitor, Traffic Logs
* Security Settings, Two-Factor Authentication
* Troubleshooting, FAQs

#### Architecture Documentation

* Frontend and Backend Architecture
* Login and Broker Login Flow (Module 03)
* Cache Architecture (Module 04)
* Security Architecture (Module 05)
* WebSockets Architecture (Module 06)
* Sandbox Architecture (Module 07)
* REST API Documentation (Module 09)
* Flow Architecture (Module 10)
* MCP Architecture (Module 41)
* Action Center (Module 42)

#### API Documentation

* All REST endpoints documented
* OpenAlgo symbol format reference
* Manual testing guide
* Bruno collections for all APIs

#### PRD Documents

* Sandbox PRD
* Python Strategies PRD
* Historify PRD
* Broker Factory Design
* WebSocket Guide
* Latency Audit

#### Other Documentation

* Why Build with OpenAlgo guide
* Ubuntu Server deployment
* Docker deployment guide
* Security Policy
* Contributor guidelines for /frontend/dist

***

### Infrastructure

#### Database Architecture (5 Databases)

* `db/openalgo.db` - Main database (users, orders, settings)
* `db/logs.db` - Traffic and API logs
* `db/latency.db` - Latency monitoring data
* `db/sandbox.db` - Analyzer/sandbox mode (isolated)
* `db/historify.duckdb` - Historical market data (DuckDB)

#### Server Configuration

* React frontend served via Flask blueprint
* Pre-built frontend dist for community use
* System permissions monitoring for db directories
* Ngrok ERR\_NGROK\_108 fix in debug mode
* Prevented duplicate startup messages
* Password reset fixed for React migration
* Startup log noise reduced (DEBUG level)

#### Docker

* Updated .dockerignore for React frontend
* Added db directory to permission commands
* Frontend documentation included

***

### Dependencies

#### Python

* DuckDB 1.4.3
* PyArrow 22.0.0
* FastParquet 2025.12.0
* simple-websocket 1.1.0
* Python 3.12+ required

#### Frontend

* React 19
* TypeScript 5.6
* Vite 6
* TanStack Query v5
* shadcn/ui components
* Tailwind CSS 4 + DaisyUI
* Biome.js
* Vitest + Playwright
* CodeMirror 6
* Socket.IO Client

***

### Breaking Changes

* Frontend routes served from React SPA
* Old Jinja2 templates removed completely
* Static folder cleaned up (React has all assets)
* API responses updated for React JSON format
* Broker login returns JSON instead of HTML redirects
* Environment variables feature removed from Python strategies

***

### Migration Guide

For users upgrading from v1.0.0.41:

1. **Backup your data**
   * Export databases before upgrading
   * Backup .env configuration
2. **Update environment**
   * Python 3.12+ required
   * Node.js 20+ for frontend development
3.  **Install dependencies**

    ```bash
    uv sync                    # Python dependencies
    cd frontend && npm install # Frontend (for development only)
    ```
4. **Database migration**
   * Existing databases are compatible
   * New sandbox.db created automatically
   * New historify.duckdb created automatically
5. **Clear browser cache**
   * React frontend requires fresh load
   * Clear all cookies and cache for the domain
6. **Review breaking changes**
   * Update any custom integrations using old template routes
   * Update broker login handling if using custom flows

***

### Contributors

Special thanks to all contributors who made this release possible:

* @Kalaiviswa - Flow Visual Builder, React migration
* @akhandhediya - WebSocket Playground
* Community contributors and testers

***

### Previous Releases

#### \[1.0.0.41] and earlier

See [GitHub Releases](https://github.com/marketcalls/openalgo/releases) for previous version history.

***

### Links

* **Repository**: https://github.com/marketcalls/openalgo
* **Documentation**: https://docs.openalgo.in
* **Discord**: https://www.openalgo.in/discord
* **YouTube**: https://www.youtube.com/@openalgo
