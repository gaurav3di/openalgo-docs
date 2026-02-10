# Excel

## OpenAlgo Excel Documentation

### Introduction

OpenAlgo provides seamless integration with Excel for executing trading strategies, fetching market data, and managing orders. This documentation covers the available Excel functions, their usage, and examples.



## Install the OpenAlgo - Excel Addins

Before installing OpenAlgo Excel Add-In, **ensure you are selecting the correct version** based on your Excel installation.

#### **📌 Steps to Check Your Excel Version**

1. **Open Microsoft Excel**
2. Click **File** → **Account**
3. Click **About Excel**
4. Look for **"32-bit"** or **"64-bit"** in the version details.

#### **🔹 Which Version Should You Install?**

* **If your Excel version is 64-bit → Install the 64-bit add-in** ✅ _(Recommended)_
* **If your Excel version is 32-bit → Install the 32-bit add-in** ✅

**Download the OpenAlgo Excel Addin**



{% embed url="https://github.com/marketcalls/OpenAlgo-Excel/releases" %}

### **⚠ .NET 6 Desktop Runtime is Required**

OpenAlgo Excel Add-In is built using **Excel-DNA**, which requires the **.NET 6 Desktop Runtime** to run add-ins.

🔹 **If the add-in is not working or Excel does not recognize it, install the .NET 6 Desktop Runtime** from the link below:

➡ [**Download .NET 6 Desktop Runtime**](https://dotnet.microsoft.com/en-us/download/dotnet/6.0)

✅ **After installing the runtime, restart your system and try loading the add-in again.**

***



#### Configuration



**Setting API Key, Version, and Host URL**



**Function:** `oa_api(api_key, [version], [host_url])`

This function must be called first to configure the API connection. All other functions use these stored credentials.

**Parameters:**

| Parameter  | Required | Default                   | Description                |
| ---------- | -------- | ------------------------- | -------------------------- |
| `api_key`  | Yes      | -                         | API key for authentication |
| `version`  | No       | `"v1"`                    | API version                |
| `host_url` | No       | `"http://127.0.0.1:5000"` | OpenAlgo server URL        |

**Example Usage:**

```
=oa_api("your_api_key")
=oa_api("your_api_key", "v1", "http://127.0.0.1:5000")
```

***

#### Account Functions



**Retrieve Funds**



**Function:** `oa_funds()`

**Example Usage:**

```
=oa_funds()
```

**Returns:** A table with available funds and margin details (available cash, collateral, M2M realized/unrealized, utilised debits).

***

**Retrieve Order Book**



**Function:** `oa_orderbook()`

**Example Usage:**

```
=oa_orderbook()
```

**Returns:** A table with all orders placed during the session.

***

**Retrieve Trade Book**



**Function:** `oa_tradebook()`

**Example Usage:**

```
=oa_tradebook()
```

**Returns:** A table with all executed trades.

***

**Retrieve Position Book**



**Function:** `oa_positionbook()`

**Example Usage:**

```
=oa_positionbook()
```

**Returns:** A table with all open positions.

***

**Retrieve Holdings**



**Function:** `oa_holdings()`

**Example Usage:**

```
=oa_holdings()
```

**Returns:** A table with all holdings in the demat account.

***

#### Market Data Functions



**Get Market Quotes**



**Function:** `oa_quotes(symbol, exchange)`

**Parameters:**

| Parameter  | Required | Description                         |
| ---------- | -------- | ----------------------------------- |
| `symbol`   | Yes      | Trading symbol                      |
| `exchange` | Yes      | Exchange (e.g., NSE, BSE, NFO, MCX) |

**Example Usage:**

```
=oa_quotes("RELIANCE", "NSE")
```

**Returns:** Market price details for the given symbol (LTP, open, high, low, close, volume, etc.).

***

**Get Market Depth**



**Function:** `oa_depth(symbol, exchange)`

**Parameters:**

| Parameter  | Required | Description    |
| ---------- | -------- | -------------- |
| `symbol`   | Yes      | Trading symbol |
| `exchange` | Yes      | Exchange       |

**Example Usage:**

```
=oa_depth("RELIANCE", "NSE")
```

**Returns:** Order book depth showing buy/sell price levels with quantity.

***

**Fetch Historical Data**



**Function:** `oa_history(symbol, exchange, interval, start_date, end_date)`

**Parameters:**

| Parameter    | Required | Description                                             |
| ------------ | -------- | ------------------------------------------------------- |
| `symbol`     | Yes      | Trading symbol                                          |
| `exchange`   | Yes      | Exchange                                                |
| `interval`   | Yes      | Candle interval (e.g., `"1m"`, `"5m"`, `"15m"`, `"1d"`) |
| `start_date` | Yes      | Start date in `YYYY-MM-DD` format                       |
| `end_date`   | Yes      | End date in `YYYY-MM-DD` format                         |

**Example Usage:**

```
=oa_history("RELIANCE", "NSE", "1m", "2024-12-01", "2024-12-31")
```

**Returns:** Historical OHLCV (Open, High, Low, Close, Volume) data in a table format.

***

**Get Supported Intervals**



**Function:** `oa_intervals()`

**Example Usage:**

```
=oa_intervals()
```

**Returns:** A list of all supported candle intervals for historical data.

***

#### Order Functions



**Place an Order**



**Function:** `oa_placeorder(strategy, symbol, action, exchange, pricetype, product, [quantity], [price], [trigger_price], [disclosed_quantity])`

**Parameters:**

| Parameter            | Required | Description                             |
| -------------------- | -------- | --------------------------------------- |
| `strategy`           | Yes      | Strategy name                           |
| `symbol`             | Yes      | Trading symbol                          |
| `action`             | Yes      | `"BUY"` or `"SELL"`                     |
| `exchange`           | Yes      | Exchange (e.g., NSE, BSE, NFO, MCX)     |
| `pricetype`          | Yes      | `"MARKET"`, `"LIMIT"`, `"SL"`, `"SL-M"` |
| `product`            | Yes      | `"MIS"`, `"CNC"`, `"NRML"`              |
| `quantity`           | No       | Number of shares/lots                   |
| `price`              | No       | Limit price (required for LIMIT orders) |
| `trigger_price`      | No       | Trigger price (required for SL orders)  |
| `disclosed_quantity` | No       | Disclosed quantity                      |

**Example Usage:**

```
=oa_placeorder("MyStrategy", "INFY", "BUY", "NSE", "LIMIT", "MIS", 10, 1500, 0, 0)
```

***

**Place a Smart Order**



**Function:** `oa_placesmartorder(strategy, symbol, action, exchange, pricetype, product, [quantity], [position_size], [price], [trigger_price], [disclosed_quantity])`

Smart orders automatically manage position sizing based on the current position.

**Parameters:**

| Parameter            | Required | Description                             |
| -------------------- | -------- | --------------------------------------- |
| `strategy`           | Yes      | Strategy name                           |
| `symbol`             | Yes      | Trading symbol                          |
| `action`             | Yes      | `"BUY"` or `"SELL"`                     |
| `exchange`           | Yes      | Exchange                                |
| `pricetype`          | Yes      | `"MARKET"`, `"LIMIT"`, `"SL"`, `"SL-M"` |
| `product`            | Yes      | `"MIS"`, `"CNC"`, `"NRML"`              |
| `quantity`           | No       | Number of shares/lots                   |
| `position_size`      | No       | Target position size                    |
| `price`              | No       | Limit price                             |
| `trigger_price`      | No       | Trigger price                           |
| `disclosed_quantity` | No       | Disclosed quantity                      |

**Example Usage:**

```
=oa_placesmartorder("SmartStrat", "INFY", "BUY", "NSE", "MARKET", "MIS", 10, "", 0, 0, 0)
```

***

**Place a Basket Order**



**Function:** `oa_basketorder(strategy, orders)`

Place multiple orders at once by referencing an Excel range containing order details.

**Parameters:**

| Parameter  | Required | Description                                 |
| ---------- | -------- | ------------------------------------------- |
| `strategy` | Yes      | Strategy name                               |
| `orders`   | Yes      | Excel range reference containing order rows |

**Example Usage:**

```
=oa_basketorder("MyStrategy", A2:J5)
```

The referenced range should contain columns for: symbol, action, exchange, pricetype, product, quantity, price, trigger\_price, disclosed\_quantity.

***

**Place a Split Order**



**Function:** `oa_splitorder(strategy, symbol, action, exchange, [quantity], [splitsize], [pricetype], [product], [price], [trigger_price], [disclosed_quantity])`

Splits a large order into smaller chunks to reduce market impact.

**Parameters:**

| Parameter            | Required | Default    | Description              |
| -------------------- | -------- | ---------- | ------------------------ |
| `strategy`           | Yes      | -          | Strategy name            |
| `symbol`             | Yes      | -          | Trading symbol           |
| `action`             | Yes      | -          | `"BUY"` or `"SELL"`      |
| `exchange`           | Yes      | -          | Exchange                 |
| `quantity`           | No       | -          | Total order quantity     |
| `splitsize`          | No       | -          | Size of each split chunk |
| `pricetype`          | No       | `"MARKET"` | Order type               |
| `product`            | No       | `"MIS"`    | Product type             |
| `price`              | No       | -          | Limit price              |
| `trigger_price`      | No       | -          | Trigger price            |
| `disclosed_quantity` | No       | -          | Disclosed quantity       |

**Example Usage:**

```
=oa_splitorder("MyStrategy", "RELIANCE", "BUY", "NSE", 100, 10, "MARKET", "MIS", 0, 0, 0)
```

***

**Modify an Order**



**Function:** `oa_modifyorder(strategy, orderid, symbol, action, exchange, [quantity], [pricetype], [product], [price], [trigger_price], [disclosed_quantity])`

**Parameters:**

| Parameter            | Required | Default    | Description            |
| -------------------- | -------- | ---------- | ---------------------- |
| `strategy`           | Yes      | -          | Strategy name          |
| `orderid`            | Yes      | -          | Order ID to modify     |
| `symbol`             | Yes      | -          | Trading symbol         |
| `action`             | Yes      | -          | `"BUY"` or `"SELL"`    |
| `exchange`           | Yes      | -          | Exchange               |
| `quantity`           | No       | -          | New quantity           |
| `pricetype`          | No       | `"MARKET"` | New order type         |
| `product`            | No       | `"MIS"`    | New product type       |
| `price`              | No       | -          | New limit price        |
| `trigger_price`      | No       | -          | New trigger price      |
| `disclosed_quantity` | No       | -          | New disclosed quantity |

**Example Usage:**

```
=oa_modifyorder("MyStrategy", "241700000023457", "RELIANCE", "BUY", "NSE", 1, "LIMIT", "MIS", 2500, 0, 0)
```

***

**Cancel an Order**



**Function:** `oa_cancelorder(strategy, orderid)`

**Parameters:**

| Parameter  | Required | Description        |
| ---------- | -------- | ------------------ |
| `strategy` | Yes      | Strategy name      |
| `orderid`  | Yes      | Order ID to cancel |

**Example Usage:**

```
=oa_cancelorder("MyStrategy", "241700000023457")
```

***

**Cancel All Orders**



**Function:** `oa_cancelallorder(strategy)`

Cancels all open orders for the given strategy.

**Parameters:**

| Parameter  | Required | Description   |
| ---------- | -------- | ------------- |
| `strategy` | Yes      | Strategy name |

**Example Usage:**

```
=oa_cancelallorder("MyStrategy")
```

***

**Close All Open Positions**



**Function:** `oa_closeposition(strategy)`

**Parameters:**

| Parameter  | Required | Description   |
| ---------- | -------- | ------------- |
| `strategy` | Yes      | Strategy name |

**Example Usage:**

```
=oa_closeposition("MyStrategy")
```

***

**Get Order Status**



**Function:** `oa_orderstatus(strategy, orderid)`

**Parameters:**

| Parameter  | Required | Description       |
| ---------- | -------- | ----------------- |
| `strategy` | Yes      | Strategy name     |
| `orderid`  | Yes      | Order ID to check |

**Example Usage:**

```
=oa_orderstatus("MyStrategy", "241700000023457")
```

***

**Get Open Position**



**Function:** `oa_openposition(strategy, symbol, exchange, product)`

**Parameters:**

| Parameter  | Required | Description                   |
| ---------- | -------- | ----------------------------- |
| `strategy` | Yes      | Strategy name                 |
| `symbol`   | Yes      | Trading symbol                |
| `exchange` | Yes      | Exchange                      |
| `product`  | Yes      | Product type (MIS, CNC, NRML) |

**Example Usage:**

```
=oa_openposition("MyStrategy", "INFY", "NSE", "MIS")
```

***

#### WebSocket Functions (Real-Time Streaming)



WebSocket functions provide real-time streaming market data. You must first connect to the WebSocket server before using any streaming functions.

**Connect to WebSocket**



**Function:** `oa_ws_connect([url])`

**Parameters:**

| Parameter | Required | Default                 | Description          |
| --------- | -------- | ----------------------- | -------------------- |
| `url`     | No       | `"ws://127.0.0.1:8765"` | WebSocket server URL |

**Example Usage:**

```
=oa_ws_connect()
=oa_ws_connect("ws://127.0.0.1:8765")
```

***

**Get Connection Status**



**Function:** `oa_ws_status()`

**Example Usage:**

```
=oa_ws_status()
```

**Returns:** Current WebSocket connection status.

***

**LTP (Last Traded Price) - Streaming**



**Function:** `oa_ws_ltp(symbol, exchange)`

**Parameters:**

| Parameter  | Required | Description    |
| ---------- | -------- | -------------- |
| `symbol`   | Yes      | Trading symbol |
| `exchange` | Yes      | Exchange       |

**Example Usage:**

```
=oa_ws_ltp("RELIANCE", "NSE")
```

**Returns:** Real-time last traded price that updates automatically.

***

**Quotes - Streaming**



**Function:** `oa_ws_quote(symbol, exchange)`

**Parameters:**

| Parameter  | Required | Description    |
| ---------- | -------- | -------------- |
| `symbol`   | Yes      | Trading symbol |
| `exchange` | Yes      | Exchange       |

**Example Usage:**

```
=oa_ws_quote("RELIANCE", "NSE")
```

**Returns:** Real-time market quote details (LTP, open, high, low, close, volume, etc.) that update automatically.

***

**Depth - Streaming**



**Function:** `oa_ws_depth(symbol, exchange, [depth_level])`

**Parameters:**

| Parameter     | Required | Description                    |
| ------------- | -------- | ------------------------------ |
| `symbol`      | Yes      | Trading symbol                 |
| `exchange`    | Yes      | Exchange                       |
| `depth_level` | No       | Number of depth levels to show |

**Example Usage:**

```
=oa_ws_depth("RELIANCE", "NSE")
=oa_ws_depth("RELIANCE", "NSE", 5)
```

**Returns:** Real-time order book depth for buy/sell levels that updates automatically.

***

**Subscribe to WebSocket Feed**



**Function:** `oa_ws_subscribe(symbol, exchange, mode, [depth_level])`

Manually subscribe to a specific data feed.

**Parameters:**

| Parameter     | Required | Description                                 |
| ------------- | -------- | ------------------------------------------- |
| `symbol`      | Yes      | Trading symbol                              |
| `exchange`    | Yes      | Exchange                                    |
| `mode`        | Yes      | Subscription mode (1=LTP, 2=Quote, 3=Depth) |
| `depth_level` | No       | Number of depth levels                      |

**Example Usage:**

```
=oa_ws_subscribe("RELIANCE", "NSE", 1)
=oa_ws_subscribe("RELIANCE", "NSE", 3, 5)
```

***

**Unsubscribe from WebSocket Feed**



**Functions:**

```
=oa_ws_unsubscribe("RELIANCE", "NSE", 1)       // Unsubscribe specific mode
=oa_ws_unsubscribe_ltp("RELIANCE", "NSE")       // Unsubscribe LTP
=oa_ws_unsubscribe_quote("RELIANCE", "NSE")     // Unsubscribe Quote
=oa_ws_unsubscribe_depth("RELIANCE", "NSE")     // Unsubscribe Depth
=oa_ws_unsubscribe_all()                         // Unsubscribe all feeds
```

***

**View Active Subscriptions**



**Function:** `oa_ws_subscriptions()`

**Example Usage:**

```
=oa_ws_subscriptions()
```

**Returns:** A table of all active WebSocket subscriptions.

***

**Get Specific Field from WebSocket Data**



**Function:** `oa_ws_field(symbol, exchange, field, [mode])`

Retrieve a specific data field from the WebSocket stream.

**Parameters:**

| Parameter  | Required | Description                                                  |
| ---------- | -------- | ------------------------------------------------------------ |
| `symbol`   | Yes      | Trading symbol                                               |
| `exchange` | Yes      | Exchange                                                     |
| `field`    | Yes      | Field name to retrieve (e.g., `"ltp"`, `"volume"`, `"open"`) |
| `mode`     | No       | Data mode                                                    |

**Example Usage:**

```
=oa_ws_field("RELIANCE", "NSE", "ltp")
=oa_ws_field("RELIANCE", "NSE", "volume", 2)
```

***

**Debug WebSocket Data**



**Function:** `oa_ws_debug(symbol, exchange, mode)`

Shows raw WebSocket data for debugging purposes.

**Parameters:**

| Parameter  | Required | Description                         |
| ---------- | -------- | ----------------------------------- |
| `symbol`   | Yes      | Trading symbol                      |
| `exchange` | Yes      | Exchange                            |
| `mode`     | Yes      | Data mode (1=LTP, 2=Quote, 3=Depth) |

**Example Usage:**

```
=oa_ws_debug("RELIANCE", "NSE", 2)
```

**Returns:** Raw data received from the WebSocket server for the given symbol and mode.

***

#### Notes



* Always call `=oa_api("your_api_key")` first to configure the API connection before using any other functions
* Test in **OpenAlgo Analyzer Mode** before using in live markets
* WebSocket functions require the OpenAlgo WebSocket server to be running (default: `ws://127.0.0.1:8765`)
* All order functions use `strategy` as the first parameter for consistency
