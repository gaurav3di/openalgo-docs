# Websockets

**WebSockets in OpenAlgo – Technical Overview**

OpenAlgo supports real-time market data streaming through WebSocket connections. This interface is designed for low-latency delivery of tick-level information across multiple asset classes.

**WebSocket URL**

To initiate a WebSocket connection with an OpenAlgo server, use:

```
ws://<host>:8765
```

For local development, the host is typically `127.0.0.1`. This URL is separate from the REST API endpoint.

**Authentication**

Immediately after establishing a WebSocket connection, the client must authenticate by sending the API key. Successful authentication is confirmed by a JSON response indicating status.

**Data Modes**

OpenAlgo WebSocket supports multiple data modes, each serving different trading and monitoring needs. The client specifies the desired mode during subscription.

| Mode | Data Type | Description                           |
| ---- | --------- | ------------------------------------- |
| 1    | LTP       | Last Traded Price only                |
| 2    | Quotes    | OHLC (Open, High, Low, Close) and LTP |
| 3    | Depth     | Top 5 market depth (bid and ask book) |

**Data Formats by Mode**

All streaming updates follow a consistent structure with the following top-level keys:

* `type`: Always "market\_data"
* `symbol`: OpenAlgo-compliant symbol
* `exchange`: Exchange code (e.g., NSE, BSE, NFO, BFO, MCX, CDS)
* `mode`: Data mode (1, 2, or 3)
* `data`: Payload specific to the selected mode

**Mode 1: LTP Format**

```json
{
  "type": "market_data",
  "symbol": "RELIANCE",
  "exchange": "NSE",
  "mode": 1,
  "data": {
    "ltp": 1424.0,
    "timestamp": 1748353658856,
    "ltt": 1748353658000
  }
}
```

**Mode 2: Quotes Format**

```json
{
  "type": "market_data",
  "symbol": "RELIANCE",
  "exchange": "NSE",
  "mode": 2,
  "data": {
    "open": 1415.0,
    "high": 1432.5,
    "low": 1408.0,
    "close": 1418.0,
    "ltp": 1424.0,
    "timestamp": 1748353824355
  }
}
```

**Mode 3: Market Depth Format**

```json
{
  "type": "market_data",
  "symbol": "RELIANCE",
  "exchange": "NSE",
  "mode": 3,
  "data": {
    "bids": [
      { "price": 1423.9, "quantity": 50 },
      { "price": 1423.5, "quantity": 35 },
      { "price": 1423.0, "quantity": 42 },
      { "price": 1422.5, "quantity": 28 },
      { "price": 1422.0, "quantity": 33 }
    ],
    "asks": [
      { "price": 1424.1, "quantity": 47 },
      { "price": 1424.5, "quantity": 39 },
      { "price": 1425.0, "quantity": 41 },
      { "price": 1425.5, "quantity": 32 },
      { "price": 1426.0, "quantity": 30 }
    ],
    "timestamp": 1748353658493
  }
}

```

**Subscription and Polling**

Subscriptions must specify both the symbol and exchange. The client receives push updates as and when market events occur. In addition, the client may periodically poll using built-in methods like `get_ltp()`, `get_quotes()`, or `get_depth()` for the latest cached values.

**Use Cases**

* Mode 1 is suitable for lightweight streaming where only LTP is needed.
* Mode 2 is useful for trading dashboards, signal generation, or OHLC analytics.
* Mode 3 is necessary for order book visualization or depth-based strategy decisions.

**Conclusion**

The OpenAlgo WebSocket feed provides a reliable and structured method for receiving real-time trading data. Proper mode selection and parsing allow efficient integration into trading algorithms and monitoring systems.
