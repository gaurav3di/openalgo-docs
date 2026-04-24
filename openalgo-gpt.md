# OpenAlgo GPT

**OpenAlgo GPT** is a Custom GPT built to help traders, developers, and algo trading enthusiasts understand OpenAlgo faster and build trading strategies more efficiently.

<figure><img src=".gitbook/assets/openalgo customGPT.png" alt=""><figcaption></figcaption></figure>

You can access it here:

[https://chatgpt.com/g/g-WK6vMYtbS-openalgo/](https://chatgpt.com/g/g-WK6vMYtbS-openalgo/)

OpenAlgo GPT is designed as an AI coding and documentation assistant for the OpenAlgo ecosystem. It can help you explore OpenAlgo documentation, understand API usage, create trading strategies, generate Python code, work with indicators, build backtests, use WebSocket market data, and prepare strategies that can be hosted inside OpenAlgo.

***

### What is OpenAlgo GPT?

OpenAlgo GPT is an AI assistant trained with OpenAlgo documentation, examples, symbol formats, order constants, indicator references, WebSocket formats, and Python strategy hosting workflows.

Instead of manually searching through multiple documentation pages, you can describe what you want to build in simple words.

For example:

```
Create a Python strategy using EMA crossover on NIFTY 5 minute candles.
Add stop loss, target, and avoid duplicate orders.
```

OpenAlgo GPT can help convert that idea into structured Python code using OpenAlgo APIs and best practices.

***

### What Can OpenAlgo GPT Help With?

OpenAlgo GPT can assist with:

* Building Python trading strategies
* Creating indicator-based strategies
* Writing backtesting scripts
* Generating Plotly candlestick charts
* Creating TradingView Lightweight Charts examples
* Fetching historical data using OpenAlgo
* Using OpenAlgo technical indicators
* Working with WebSocket live market data
* Creating live or paper trading bots
* Understanding symbol formats
* Understanding order constants
* Placing, modifying, and cancelling orders
* Debugging OpenAlgo API code
* Handling broker integration questions
* Preparing strategies for OpenAlgo Python Strategy Hosting
* Using APScheduler for scheduled strategies
* Using SQLAlchemy for database-related workflows when required

***

### Strategy Development Assistance

OpenAlgo GPT can help users move from a strategy idea to working code.

You can ask for strategies such as:

```
Build an RSI strategy for SBIN using 5 minute candles.
Buy when RSI crosses above 30 and sell when RSI crosses below 70.
Add stop loss and target.
```

```
Create a Supertrend strategy using OpenAlgo indicators.
Use historical data from start_date and end_date.
Add duplicate order prevention.
```

```
Build a VWAP intraday strategy with market hours check and square-off logic.
```

OpenAlgo GPT can help structure the code with:

* OpenAlgo client initialization
* Historical data fetching
* DataFrame validation
* Indicator calculation
* Signal generation
* Entry and exit logic
* Risk management
* Order placement
* Order response validation
* Duplicate-order prevention
* Graceful shutdown

***

### OpenAlgo Python SDK Support

OpenAlgo GPT can help users understand and use the OpenAlgo Python SDK.

It can assist with:

* Installing the OpenAlgo Python package
* Initializing the OpenAlgo client
* Fetching historical data
* Fetching quotes
* Fetching market depth
* Placing market orders
* Placing limit orders
* Using smart orders
* Using basket orders
* Using split orders
* Modifying orders
* Cancelling orders
* Checking positions
* Checking holdings
* Understanding API responses

Example prompt:

```
Show me how to place a market order in OpenAlgo using Python.
```

***

### Historical Data and DataFrame Handling

OpenAlgo GPT follows OpenAlgo historical data conventions.

It can help generate code using:

```python
df = client.history(
    symbol="SBIN",
    exchange="NSE",
    interval="5m",
    start_date="2025-04-01",
    end_date="2025-04-08"
)
```

For daily candles, OpenAlgo GPT uses:

```python
interval="D"
```

instead of:

```python
interval="1d"
```

It can also help validate historical data before use:

* Check if the DataFrame is empty
* Check required OHLCV columns
* Sort candles
* Handle missing values
* Drop NaN values after indicator calculation

***

### OpenAlgo Technical Indicators

OpenAlgo GPT can help users work with OpenAlgo’s technical indicator library.

It can assist with indicator categories such as:

#### Trend Indicators

* SMA
* EMA
* WMA
* HMA
* Supertrend
* Ichimoku
* Alligator
* Moving Average Envelopes

#### Momentum Indicators

* RSI
* MACD
* Stochastic
* CCI
* Williams %R
* Fisher Transform
* Connors RSI

#### Volatility Indicators

* ATR
* Bollinger Bands
* Keltner Channel
* Donchian Channel
* Chandelier Exit
* Historical Volatility

#### Volume Indicators

* OBV
* VWAP
* MFI
* ADL
* CMF
* RVOL

#### Statistical Indicators

* Linear Regression
* Linear Regression Slope
* Correlation
* Beta
* Variance
* Z-Score

#### Hybrid Indicators

* ADX
* Aroon
* Pivot Points
* Parabolic SAR

#### Utility Functions

* Crossover
* Crossunder
* Cross
* Highest
* Lowest
* Change
* Rate of Change

Example prompt:

```
Create a strategy using EMA 20, EMA 50, RSI, and OpenAlgo crossover functions.
```

***

### Backtesting Support

OpenAlgo GPT can help users create simple and practical backtesting scripts.

It can generate backtests with:

* Historical data fetch
* Indicator calculation
* Signal generation
* Entry logic
* Exit logic
* Position tracking
* Trade list
* Profit and loss calculation
* Win rate
* Net P\&L
* Drawdown
* Summary statistics

Example prompt:

```
Backtest an EMA crossover strategy on NIFTY using OpenAlgo historical data.
Show total trades, win rate, net P&L, and drawdown.
```

Backtesting examples are for research and educational purposes. Historical results do not guarantee future performance.

***

### Live and Paper Trading Strategy Support

OpenAlgo GPT can help convert strategy logic into live or paper trading scripts.

It can include practical safety controls such as:

* Quantity or capital control
* Stop loss
* Target
* Max trades per day
* Duplicate-order prevention
* Candle-close confirmation
* Market-hours checks
* Position checks
* Open order checks
* Safe shutdown handling

Example prompt:

```
Convert this backtest into a paper trading strategy using OpenAlgo.
Add duplicate order prevention and print all order responses.
```

For live trading, users should test in paper mode or a controlled environment before using real capital.

***

### WebSocket Market Data Support

OpenAlgo GPT can help users work with OpenAlgo WebSocket feeds.

It supports examples for:

* LTP streaming
* Quote streaming
* Depth streaming
* WebSocket authentication
* Subscription format
* Unsubscription format
* Callback-based data handling
* Verbose mode control
* Graceful disconnect

OpenAlgo WebSocket modes:

```
Mode 1: LTP
Mode 2: Quote
Mode 3: Depth
```

Example prompt:

```
Create a Python WebSocket script to subscribe to NIFTY and INFY quote data and print LTP updates.
```

***

### Plotly and Data Visualization

OpenAlgo GPT can help create charts for market data and indicators.

It can generate:

* Plotly candlestick charts
* OHLC charts
* Indicator overlays
* Volume charts
* Backtest equity curves
* Signal markers
* Strategy performance charts

For Plotly candlestick charts, OpenAlgo GPT follows the working Plotly example and uses:

```python
fig.update_xaxes(type="category")
```

This helps avoid gaps in candle charts caused by non-trading periods.

Example prompt:

```
Create a Plotly candlestick chart with EMA 20, EMA 50, buy signals, and sell signals.
```

***

### OpenAlgo Symbol Format Support

OpenAlgo GPT can help users understand the OpenAlgo symbol format across different instruments.

It can assist with:

* Equity symbols
* Index symbols
* Futures symbols
* Options symbols
* Currency symbols
* Commodity symbols

Examples:

```
SBIN
INFY
RELIANCE
```

Futures format:

```
[Base Symbol][Expiration Date]FUT
```

Options format:

```
[Base Symbol][Expiration Date][Strike Price][Option Type]
```

Example:

```
NIFTY28MAR2420800CE
```

***

### Order Constants Support

OpenAlgo GPT can help users use the correct OpenAlgo order constants.

#### Exchanges

```
NSE
NFO
CDS
BSE
BFO
BCD
MCX
NCDEX
NSE_INDEX
BSE_INDEX
```

#### Product Types

```
CNC
NRML
MIS
```

#### Price Types

```
MARKET
LIMIT
SL
SL-M
```

#### Actions

```
BUY
SELL
```

Example prompt:

```
Explain which exchange and product type I should use for NIFTY options intraday trading.
```

***

### Python Strategy Hosting Inside OpenAlgo

OpenAlgo GPT can help users prepare strategies that can be hosted inside OpenAlgo itself.

OpenAlgo’s Python Strategy Hosting system allows users to:

* Upload Python strategy files
* Start and stop strategies
* Schedule strategies
* Select exchange-specific calendars
* Pass custom parameters
* Monitor strategy output
* Run each strategy in an isolated process

Example prompt:

```
Create a Python strategy that can be uploaded to OpenAlgo Strategy Hosting.
Read SYMBOL, EXCHANGE, API key, and host from environment variables.
```

Hosted strategies can read environment variables such as:

```python
OPENALGO_API_KEY
HOST_SERVER
OPENALGO_HOST
OPENALGO_STRATEGY_EXCHANGE
WEBSOCKET_URL
WEBSOCKET_HOST
WEBSOCKET_PORT
```

For hosted strategies, OpenAlgo GPT encourages using:

```python
EXCHANGE = os.getenv(
    "OPENALGO_STRATEGY_EXCHANGE",
    os.getenv("EXCHANGE", "NSE")
)
```

This helps keep the script’s order exchange aligned with the exchange calendar selected in OpenAlgo Strategy Hosting.

***

### Scheduler Support

OpenAlgo GPT can help users build scheduled strategies using APScheduler.

For Indian market strategies, it uses IST timezone with `pytz`.

Example:

```python
IST = pytz.timezone("Asia/Kolkata")
```

It can help create scheduled jobs for:

* Running strategies at fixed intervals
* Starting at market open
* Stopping near market close
* Periodic signal checks
* Scheduled square-off logic

Example prompt:

```
Create an APScheduler strategy that checks signals every 5 minutes in IST.
```

***

### Database Support

OpenAlgo GPT does not write to a database unless the user asks.

If database interaction is required, it uses SQLAlchemy.

It can help with:

* Storing trades
* Storing signals
* Storing backtest results
* Creating SQLAlchemy models
* Creating database sessions
* Reading and writing trade records safely

Example prompt:

```
Store my executed trades in a SQLite database using SQLAlchemy.
```

***

### Practical Safety Practices

OpenAlgo GPT encourages practical strategy safety.

It can help add:

* Stop loss
* Target
* Trailing stop
* Position sizing
* Max trades per day
* Candle-close confirmation
* Duplicate-order prevention
* Order response validation
* Graceful shutdown
* Market-hours checks
* Paper trading mode

It avoids unsafe patterns such as:

* Repeated order placement on the same candle
* Repeated order placement on the same signal
* Blind live trading loops
* Hardcoded API keys
* Unchecked order responses
* Mixing backtest logic with live execution without clear separation

***

### Example Prompts

You can try prompts like:

```
Create a Python strategy using OpenAlgo for EMA crossover on SBIN.
Use 5 minute candles, start_date and end_date, and add stop loss and target.
```

```
Build a backtest for RSI 30/70 strategy using OpenAlgo historical data.
Show total trades, win rate, net P&L, and drawdown.
```

```
Create a Plotly candlestick chart using OpenAlgo data with EMA 20 and EMA 50.
```

```
Create a WebSocket quote streaming script for NIFTY and INFY.
Print all incoming LTP updates.
```

```
Create a Python strategy suitable for OpenAlgo Strategy Hosting.
Read parameters from environment variables.
```

```
Explain the correct OpenAlgo symbol format for NIFTY options.
```

```
Generate a Supertrend strategy using OpenAlgo indicators with duplicate-order prevention.
```

```
Convert this strategy into a paper trading bot.
```

***

### Who Is OpenAlgo GPT For?

OpenAlgo GPT is useful for:

* Algo traders
* Python developers
* Quant learners
* Trading system builders
* OpenAlgo users
* Strategy developers
* Backtesting enthusiasts
* Traders learning automation
* Developers building broker-integrated workflows

Whether you are writing your first Python strategy or improving a more advanced trading system, OpenAlgo GPT can help you move faster.

***

### Best Practices When Using OpenAlgo GPT

For better results, include details such as:

* Symbol
* Exchange
* Timeframe
* Start date and end date
* Indicator rules
* Entry condition
* Exit condition
* Quantity or capital
* Product type
* Stop loss
* Target
* Backtest or live mode
* Paper trading or real execution
* Whether it should be hosted inside OpenAlgo

Example:

```
Create a paper trading strategy for RELIANCE on NSE using 5 minute candles.
Buy when EMA 20 crosses above EMA 50.
Exit when EMA 20 crosses below EMA 50.
Use MIS product, quantity 1, stop loss 1 percent, target 2 percent.
Make it suitable for OpenAlgo Strategy Hosting.
```

***

### Community

We encourage OpenAlgo community members to explore, build, test, and share what they are working on.

Sharing strategies, examples, improvements, and experiments helps others learn and strengthens the OpenAlgo ecosystem.

Community Discord:

[https://openalgo.in/discord](https://openalgo.in/discord)

OpenAlgo Documentation:

[https://docs.openalgo.in](https://docs.openalgo.in/)

OpenAlgo GPT:

[https://chatgpt.com/g/g-WK6vMYtbS-openalgo/](https://chatgpt.com/g/g-WK6vMYtbS-openalgo/)

***

```
```

