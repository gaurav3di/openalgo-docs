# Telegram

### Overview

The OpenAlgo Telegram Bot provides a convenient interface to access your trading data through Telegram. It offers read-only access to your trading account, allowing you to view positions, orders, holdings, P\&L, and generate charts directly from your Telegram app.

{% embed url="https://www.youtube.com/watch?v=Z4nZbUiw54I" %}

### Features

* **Account Linking**: Securely link your OpenAlgo account using API keys
* **Real-time Data Access**: View orderbook, tradebook, positions, holdings, and funds
* **P\&L Tracking**: Monitor realized and unrealized profit/loss
* **Quote Information**: Get real-time quotes for any symbol
* **Chart Generation**: Generate intraday and daily charts with technical indicators
* **Interactive Menu**: Easy-to-use button interface for quick access

### Setup

#### 1. Enable Telegram Bot in OpenAlgo

1. Navigate to the Telegram section in your OpenAlgo dashboard
2. Click on "Configure Bot"
3. Enter your Telegram Bot Token (obtained from BotFather)
4. Click "Save Configuration"
5. Click "Start Bot" to activate the bot

#### 2. Create Your Telegram Bot

1. Open Telegram and search for [@BotFather](https://t.me/botfather)
2. Send `/newbot` command
3. Follow the instructions to create your bot
4. Copy the bot token provided by BotFather
5. Use this token in the OpenAlgo configuration

#### 3. Link Your Account

1. Start a conversation with your bot on Telegram
2.  Send the `/link` command with your API key and host URL:

    ```
    /link your_api_key_here http://your-openalgo-host:5000
    ```
3. The bot will confirm successful linking

### Available Commands

#### Account Management

* `/start` - Initialize the bot and see welcome message
* `/link <api_key> <host_url>` - Link your OpenAlgo account
* `/unlink` - Unlink your account
* `/status` - Check connection status

#### Trading Data

* `/orderbook` - View all orders
* `/tradebook` - View executed trades
* `/positions` - View open positions
* `/holdings` - View holdings
* `/funds` - View available funds
* `/pnl` - View profit & loss summary

#### Market Data

* `/quote <symbol> [exchange]` - Get quote for a symbol
  * Example: `/quote RELIANCE`
  * Example: `/quote NIFTY NSE_INDEX`

#### Charts

* `/chart <symbol> [exchange] [type] [interval] [days]` - Generate price charts
  * Type: `intraday` (default), `daily`, or `both`
  * Intervals: `1m`, `3m`, `5m`, `15m`, `30m`, `1h`, `D`
  * Examples:
    * `/chart RELIANCE` - 5-minute intraday chart
    * `/chart RELIANCE NSE intraday 15m 10` - 15-minute chart for 10 days
    * `/chart RELIANCE NSE daily D 100` - Daily chart for 100 days
    * `/chart RELIANCE NSE both` - Both intraday and daily charts

#### Interactive Interface

* `/menu` - Display interactive button menu
* `/help` - Show help message with all commands

### Order Alerts (Automatic Notifications)

#### Overview

The bot automatically sends real-time notifications for all order-related API activities. No additional commands are needed - alerts are sent automatically when orders are placed through the OpenAlgo API.

#### Supported Order Types

* **Place Order** - Regular order placement notifications
* **Place Smart Order** - Smart orders with position sizing alerts
* **Basket Order** - Multiple orders in one request notifications
* **Split Order** - Large orders split into smaller chunks alerts
* **Modify Order** - Order modification notifications
* **Cancel Order** - Single order cancellation alerts
* **Cancel All Orders** - Bulk order cancellation notifications
* **Close Position** - Position closing alerts

#### Alert Format

Each alert includes:

* **Mode Indicator**:
  * 💰 **LIVE MODE** - Real orders executed with the broker
  * 🔬 **ANALYZE MODE** - Simulated orders for testing/analysis
* **Order Details**: Symbol, action, quantity, price, exchange, product type
* **Status**: Success or failure with error messages if applicable
* **Order ID**: Unique identifier for tracking
* **Timestamp**: Time of order execution
* **Strategy Name**: If provided in the API call

#### Example Notifications

**Live Order Placed:**

```
📈 Order Placed
💰 LIVE MODE - Real Order
─────────────────────
Strategy: MyStrategy
Symbol: RELIANCE
Action: BUY
Quantity: 10
Price Type: MARKET
Exchange: NSE
Product: MIS
Order ID: 250408000989443
⏰ Time: 14:23:45
```

**Analyze Mode Order:**

```
📈 Order Placed
🔬 ANALYZE MODE - No Real Order
─────────────────────
Strategy: TestStrategy
Symbol: RELIANCE
Action: BUY
Quantity: 10
Price Type: MARKET
Exchange: NSE
Product: MIS
Order ID: ANALYZE123456
⏰ Time: 14:23:45
```

#### Configuration

* Alerts are **enabled by default** for all linked users
* Users can enable/disable notifications in their preferences
* Failed alerts are queued for retry when the bot comes online
* Zero impact on order execution speed (asynchronous processing)

#### Requirements for Receiving Alerts

1. Telegram bot must be running (`/telegram/bot/start` in web interface)
2. Your account must be linked via `/link` command
3. Notifications must be enabled (default: enabled)
4. Orders must be placed through the OpenAlgo API

### Chart Features

#### Intraday Charts

* Default interval: 5 minutes
* Default period: 5 days
* Includes candlestick pattern and volume bars
* No gaps for non-trading hours

#### Daily Charts

* Default interval: Daily (D)
* Default period: 252 trading days
* Includes moving averages (MA20, MA50, MA200)
* Shows candlestick patterns with volume

#### Chart Customization

* **Intervals**: 1m, 3m, 5m, 15m, 30m, 1h for intraday; D for daily
* **Days**: Customize the lookback period
* **Exchange**: Supports NSE, BSE, NFO, CDS, MCX, NSE\_INDEX, BSE\_INDEX

### Security

#### API Key Encryption

* API keys are encrypted using Fernet encryption before storage
* Keys are never stored in plain text
* Each user's API key is isolated

#### Authentication

* Bot requires valid API key for account linking
* API key is validated against OpenAlgo server
* Session-based authentication for all data requests

#### Privacy

* Each Telegram user can only access their own linked account
* No cross-user data access
* Command usage is logged for security audit

### Database Schema

The bot uses SQLAlchemy ORM with the following tables:

#### TelegramUser

* Stores user-bot linkage information
* Encrypted API key storage
* User preferences and settings

#### BotConfig

* Bot configuration settings
* Webhook URL and polling mode
* Bot state management

#### CommandLog

* Audit trail of all commands
* Usage analytics
* Error tracking

### Technical Architecture

#### Components

1. **TelegramBotService** (`services/telegram_bot_service.py`)
   * Core bot logic and command handlers
   * OpenAlgo SDK integration
   * Chart generation using Plotly
2. **Database Layer** (`database/telegram_db.py`)
   * SQLAlchemy models and queries
   * Encryption/decryption utilities
   * Configuration management
3. **Blueprint** (`blueprints/telegram.py`)
   * Flask routes for bot management
   * Web interface for configuration
   * Bot lifecycle management
4. **Auto-start Feature**
   * Bot automatically starts on application launch if previously active
   * State persistence across restarts
   * Configured in `app.py`

#### Threading Model

* Bot runs in a separate thread with its own event loop
* Non-blocking operation with main Flask application
* Graceful shutdown handling

#### Chart Generation

* Uses Plotly for chart creation
* Kaleido engine for image export
* Pandas for data manipulation
* Category-type x-axis to handle gaps

### Troubleshooting

#### Bot Not Responding

1. Check if bot is running in OpenAlgo dashboard
2. Verify bot token is correct
3. Check network connectivity
4. Review logs for errors

#### Chart Generation Issues

1. Ensure market data is available for the symbol
2. Check if the exchange is correct
3. Verify interval is supported
4. Check date range is valid

#### Linking Issues

1. Verify API key is correct
2. Ensure host URL is accessible
3. Check if API key has necessary permissions
4. Verify OpenAlgo server is running

### Environment Variables

The bot respects the following environment variables:

* `DATABASE_URL` - Database connection string
* `ENCRYPTION_KEY` - Fernet encryption key for API keys
* `APP_KEY` - Flask application secret key
* `HOST_SERVER` - OpenAlgo server URL

### API Endpoints

#### Web Interface

* `GET /telegram/` - Bot management dashboard
* `POST /telegram/config` - Update bot configuration
* `POST /telegram/start` - Start the bot
* `POST /telegram/stop` - Stop the bot
* `POST /telegram/restart` - Restart the bot

### Error Handling

* All errors are logged with context
* User-friendly error messages in Telegram
* Automatic retry for transient failures
* Graceful degradation for missing data

### Performance Considerations

* Charts are generated asynchronously
* API calls use connection pooling
* Database queries are optimized with indexes
* Image generation uses efficient Kaleido backend

### Future Enhancements

* [ ] Real-time price alerts
* [ ] Portfolio analytics
* [ ] Multi-account support
* [ ] Custom indicators on charts
* [ ] Webhook mode for better scalability
* [ ] Inline queries for quick quotes

### Support

For issues or questions:

1. Check the logs in OpenAlgo dashboard
2. Review this documentation
3. Contact OpenAlgo support
