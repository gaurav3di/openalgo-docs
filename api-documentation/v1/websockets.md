# WebSocket API

OpenAlgo exposes normalized real-time market data through a separate WebSocket proxy. It is not mounted below the REST `/api/v1` prefix.

## Connection URL

```text
Local:      ws://127.0.0.1:8765
Production: wss://<your-domain>/ws
```

The direct host and port come from `WEBSOCKET_HOST` and `WEBSOCKET_PORT`. A production reverse proxy normally exposes the service at `/ws` over TLS.

## Authenticate

Send authentication immediately after connecting. The default authentication grace period is 15 seconds.

```json
{
  "action": "authenticate",
  "api_key": "<your_app_apikey>"
}
```

`apikey` is also accepted as the key field. A successful response identifies the active broker:

```json
{
  "type": "auth",
  "status": "success",
  "message": "Authentication successful",
  "broker": "zerodha",
  "user_id": "openalgo-user",
  "supported_features": {
    "ltp": true,
    "quote": true,
    "depth": true
  }
}
```

## Subscribe

Use `LTP`, `Quote`, or `Depth`. Mode names are case-insensitive; integers `1`, `2`, and `3` are accepted respectively. The optional `depth` field defaults to 5. `request_id` is echoed in the acknowledgement when supplied.

```json
{
  "action": "subscribe",
  "mode": "Quote",
  "symbols": [
    {"exchange": "NSE", "symbol": "RELIANCE"},
    {"exchange": "NSE", "symbol": "INFY"}
  ],
  "request_id": "quotes-1"
}
```

A single `symbol` and `exchange` pair can be sent instead of `symbols`. The acknowledgement reports success or failure for each requested instrument:

```json
{
  "type": "subscribe",
  "status": "success",
  "subscriptions": [
    {
      "symbol": "RELIANCE",
      "exchange": "NSE",
      "status": "success",
      "mode": "Quote",
      "depth": 5,
      "broker": "zerodha"
    }
  ],
  "message": "Subscription processing complete",
  "broker": "zerodha",
  "request_id": "quotes-1"
}
```

## Market-Data Messages

Ticks use a common wrapper. The nested `data` object is the normalized broker payload for the requested mode.

```json
{
  "type": "market_data",
  "symbol": "RELIANCE",
  "exchange": "NSE",
  "mode": 2,
  "broker": "zerodha",
  "data": {
    "ltp": 1424.0,
    "open": 1415.0,
    "high": 1432.5,
    "low": 1408.0,
    "close": 1418.0,
    "volume": 100000
  }
}
```

| Mode | Value | Payload |
|---|---:|---|
| LTP | 1 | Last traded price and tick fields |
| Quote | 2 | LTP plus normalized OHLC and volume fields |
| Depth | 3 | Quote fields plus available bid and ask levels |

Depth availability and the number of levels depend on the active broker. Higher-mode broker ticks can satisfy subscribers to lower modes.

## Unsubscribe

For selected subscriptions, put the mode on each symbol object:

```json
{
  "action": "unsubscribe",
  "symbols": [
    {"exchange": "NSE", "symbol": "RELIANCE", "mode": "Quote"}
  ],
  "request_id": "quotes-2"
}
```

Remove every subscription owned by the connection with:

```json
{
  "action": "unsubscribe_all"
}
```

## Other Actions

| Action | Purpose |
|---|---|
| `get_broker_info` | Return the authenticated broker and adapter state |
| `get_supported_brokers` | Return brokers supported by the proxy configuration |
| `ping` | Return a `pong` with the server timestamp |

The `ping` action echoes an optional `timestamp` and `_pingId`, which can be used for latency measurement.

## Reconnection

WebSocket transport pings are configured by `WS_PING_INTERVAL` and `WS_PING_TIMEOUT`, both 20 seconds by default. After a disconnect, reconnect, authenticate again, and restore the required subscriptions.
