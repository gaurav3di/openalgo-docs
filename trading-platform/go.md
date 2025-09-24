# GO

## Go SDK for OpenAlgo

To install the OpenAlgo Go library, use go get:

```bash
go get github.com/marketcalls/openalgo-go
```

#### Get the OpenAlgo apikey

Make Sure that your OpenAlgo Application is running. Login to OpenAlgo Application with valid credentials and get the OpenAlgo apikey

For detailed function parameters refer to the [API Documentation](https://docs.openalgo.in/api-documentation/v1)

#### Getting Started with OpenAlgo

First, import the OpenAlgo package and initialize it with your API key:

```go
package main

import (
    "github.com/marketcalls/openalgo-go/openalgo"
)

// Replace 'your_api_key_here' with your actual API key
// Specify the host URL with your hosted domain or ngrok domain.
// If running locally then use the default host value.
// Parameters: apiKey, host, version, websocketURL (optional)
client := openalgo.NewClient(
    "your_api_key_here",
    "http://127.0.0.1:5000",
    "v1",                    // API version
    "ws://127.0.0.1:8765",   // WebSocket URL (optional)
)

```

#### Check OpenAlgo Version

```go
import (
    "fmt"
    "github.com/marketcalls/openalgo-go/openalgo"
)

fmt.Printf("OpenAlgo Go SDK Version: %s\n", openalgo.Version)
```

#### Complete List of Implemented Functions

**Order Management**

* `PlaceOrder` - Place a new order
* `PlaceSmartOrder` - Place a smart order with position sizing
* `BasketOrder` - Place multiple orders at once
* `SplitOrder` - Split large orders into smaller chunks
* `ModifyOrder` - Modify an existing order
* `CancelOrder` - Cancel a specific order
* `CancelAllOrder` - Cancel all pending orders
* `ClosePosition` - Close all open positions
* `OrderStatus` - Get order status
* `OpenPosition` - Get open position for a symbol

**Market Data**

* `Quotes` - Get real-time quotes
* `Depth` - Get market depth
* `History` - Get historical data
* `Intervals` - Get available time intervals
* `Symbol` - Get symbol information
* `Search` - Search for symbols
* `Expiry` - Get expiry dates

**Account Information**

* `Funds` - Get account funds
* `OrderBook` - Get all orders
* `TradeBook` - Get all trades
* `PositionBook` - Get all positions
* `Holdings` - Get holdings

**Utility**

* `Ping` - Check API connectivity
* `AnalyzerStatus` - Get analyzer status
* `AnalyzerToggle` - Toggle analyzer mode (requires boolean: true to enable, false to disable)

**WebSocket Streaming**

* `Connect` - Connect to WebSocket
* `Disconnect` - Disconnect from WebSocket
* `SubscribeLTP` - Subscribe to LTP updates
* `UnsubscribeLTP` - Unsubscribe from LTP
* `SubscribeQuote` - Subscribe to quote updates
* `UnsubscribeQuote` - Unsubscribe from quotes
* `SubscribeDepth` - Subscribe to market depth
* `UnsubscribeDepth` - Unsubscribe from depth

#### Examples

Please refer to the documentation on [order constants](https://docs.openalgo.in/api-documentation/v1/order-constants), and consult the API reference for details on optional parameters

#### Ping Example

Check API connectivity:

```go
response, err := client.Ping()
if err != nil {
    log.Printf("Error: %v", err)
}
fmt.Printf("%v\n", response)
```

**Ping Response**

```go
map[data:map[broker:shoonya message:pong] status:success]
```

#### PlaceOrder Example

To place a new market order:

```go
response, err := client.PlaceOrder(
    "GO Strategy", // strategy
    "NHPC",      // symbol
    "BUY",       // action
    "NSE",       // exchange
    "MARKET",    // price_type
    "MIS",       // product
    1,           // quantity
)
if err != nil {
    log.Printf("Error: %v", err)
}
fmt.Printf("%v\n", response)
```

**Place Market Order Response**

```go
map[orderid:25092400475638 status:success]
```

To place a new limit order:

```go
response, err := client.PlaceOrder(
    "GO Strategy", // strategy
    "YESBANK",   // symbol
    "BUY",       // action
    "NSE",       // exchange
    "LIMIT",     // price_type
    "MIS",       // product
    "1",         // quantity
    map[string]interface{}{
        "price":              16.0,
        "trigger_price":      0.0,
        "disclosed_quantity": "0",
    },
)
if err != nil {
    log.Printf("Error: %v", err)
}
fmt.Printf("%v\n", response)
```

**Place Limit Order Response**

```go
map[orderid:25092400479260 status:success]
```

#### PlaceSmartOrder Example

To place a smart order considering the current position size:

```go
response, err := client.PlaceSmartOrder(
    "GO Strategy", // strategy
    "TATAMOTORS",  // symbol
    "SELL",        // action
    "NSE",         // exchange
    "MARKET",      // price_type
    "MIS",         // product
    1,             // quantity
    5,             // position_size
)
if err != nil {
    log.Printf("Error: %v", err)
}
fmt.Printf("%v\n", response)
```

**Place Smart Market Order Response**

```go
map[orderid:25092400482144 status:success]
```

#### BasketOrder Example

To place a new basket order:

```go
basketOrders := []map[string]interface{}{
    {
        "symbol":    "BHEL",
        "exchange":  "NSE",
        "action":    "BUY",
        "quantity":  1,
        "pricetype": "MARKET",
        "product":   "MIS",
    },
    {
        "symbol":    "ZOMATO",
        "exchange":  "NSE",
        "action":    "SELL",
        "quantity":  1,
        "pricetype": "MARKET",
        "product":   "MIS",
    },
}
response, err := client.BasketOrder("GO Strategy", basketOrders)
if err != nil {
    log.Printf("Error: %v", err)
}
fmt.Printf("%v\n", response)
```

**Basket Order Response**

```go
map[results:[map[orderid:25092400496406 status:success symbol:YESBANK] map[orderid:25092400496407 status:success symbol:NHPC]] status:success]
```

#### SplitOrder Example

To place a new split order:

```go
response, err := client.SplitOrder(
    "GO Strategy", // strategy
    "YESBANK",   // symbol
    "NSE",       // exchange
    "SELL",      // action
    105,         // quantity
    20,          // split_size
    "MARKET",    // price_type
    "MIS",       // product
)
if err != nil {
    log.Printf("Error: %v", err)
}
fmt.Printf("%v\n", response)
```

**SplitOrder Response**

```go
map[results:[map[order_num:1 orderid:25092400502095 quantity:2 status:success] map[order_num:2 orderid:25092400502096 quantity:2 status:success]] split_size:2 status:success total_quantity:4]
```

#### ModifyOrder Example

To modify an existing order:

```go
response, err := client.ModifyOrder(
    "250408001002736", // order_id
    "GO Strategy",     // strategy
    "YESBANK",         // symbol
    "BUY",             // action
    "NSE",             // exchange
    "LIMIT",           // price_type
    "CNC",             // product
    1,                 // quantity
    "16.5",            // price
    "0",               // disclosed_quantity
    "0",               // trigger_price
)
if err != nil {
    log.Printf("Error: %v", err)
}
fmt.Printf("%v\n", response)
```

**Modify Order Response**

```go
map[orderid:25092400479260 status:success]
```

#### CancelOrder Example

To cancel an existing order:

```go
response, err := client.CancelOrder(
    "250408001002736", // order_id
    "GO Strategy",     // strategy
)
if err != nil {
    log.Printf("Error: %v", err)
}
fmt.Printf("%v\n", response)
```

**Cancelorder Response**

```go
map[orderid:25092400479260 status:success]
```

#### CancelAllOrder Example

To cancel all open orders and trigger pending orders:

```go
response, err := client.CancelAllOrder("GO Strategy")
if err != nil {
    log.Printf("Error: %v", err)
}
fmt.Printf("%v\n", response)
```

**Cancelallorder Response**

```go
map[canceled_orders:[25092400490404] failed_cancellations:[] message:Canceled 1 orders. Failed to cancel 0 orders. status:success]
```

#### ClosePosition Example

To close all open positions across various exchanges:

```go
response, err := client.ClosePosition("GO Strategy")
if err != nil {
    log.Printf("Error: %v", err)
}
fmt.Printf("%v\n", response)
```

**ClosePosition Response**

```go
map[message:All Open Positions Squared Off status:success]
```

#### OrderStatus Example

To Get the Current OrderStatus:

```go
response, err := client.OrderStatus(
    "250408000989443", // order_id
    "GO Strategy",     // strategy
)
if err != nil {
    log.Printf("Error: %v", err)
}
fmt.Printf("%v\n", response)
```

**Orderstatus Response**

```go
map[data:map[action:BUY average_price:21.29 exchange:NSE order_status:complete orderid:25092400496406 price:0 pricetype:MARKET product:MIS quantity:1 symbol:YESBANK timestamp:14:14:26 24-09-2025 trigger_price:0] status:success]
```

#### OpenPosition Example

To Get the Current OpenPosition:

```go
response, err := client.OpenPosition(
    "GO Strategy",   // strategy
    "YESBANK",       // symbol
    "NSE",           // exchange
    "MIS",           // product
)
if err != nil {
    log.Printf("Error: %v", err)
}
fmt.Printf("%v\n", response)
```

**OpenPosition Response**

```go
map[quantity:-3 status:success]
```

#### Quotes Example

```go
response, err := client.Quotes("RELIANCE", "NSE")
if err != nil {
    log.Printf("Error: %v", err)
}
fmt.Printf("%v\n", response)
```

**Quotes response**

```go
map[data:map[ask:1392.8 bid:1392.6 high:1396 low:1383.2 ltp:1392.8 oi:0 open:1385.3 prev_close:1389.8 volume:2.477476e+06] status:success]
```

#### Depth Example

```go
response, err := client.Depth("SBIN", "NSE")
if err != nil {
    log.Printf("Error: %v", err)
}
fmt.Printf("%v\n", response)
```

**Depth Response**

```go
map[data:map[asks:[map[price:870.65 quantity:120] map[price:870.7 quantity:111] map[price:870.75 quantity:105] map[price:870.8 quantity:69] map[price:870.85 quantity:244]] bids:[map[price:870.35 quantity:36] map[price:870.3 quantity:70] map[price:870.2 quantity:345] map[price:870.15 quantity:575] map[price:870.1 quantity:611]] high:880.4 low:869.1 ltp:870.35 ltq:1 oi:0 open:875.2 prev_close:870.5 totalbuyqty:1637 totalsellqty:649 volume:313325] status:success]
```

#### History Example

```go
response, err := client.History(
    "SBIN",       // symbol
    "NSE",        // exchange
    "5m",         // interval
    "2025-04-01", // start_date
    "2025-04-08", // end_date
)
if err != nil {
    log.Printf("Error: %v", err)
}
fmt.Printf("%v\n", response)
```

**History Response**

```go
map[data:[map[close:806 high:806.45 low:803.4 oi:0 open:803.4 timestamp:1.7566983e+09 volume:268668] map[close:807.4 high:807.85 low:805.35 oi:0 open:806.1 timestamp:1.7566986e+09 volume:205484] map[close:806.75 high:808.3 low:806.05 oi:0 open:807.5 timestamp:1.7566989e+09 volume:135774] map[close:805.85 high:807 low:805.5 oi:0 open:806.65 timestamp:1.7566992e+09 
```

#### Intervals Example

```go
response, err := client.Intervals()
if err != nil {
    log.Printf("Error: %v", err)
}
fmt.Printf("%v\n", response)
```

**Intervals response**

```go
map[data:map[days:[D] hours:[1h 2h 4h] minutes:[1m 3m 5m 10m 15m 30m] months:[] seconds:[] weeks:[]] status:success]
```

#### Symbol Example

```go
response, err := client.Symbol("RELIANCE", "NSE")
if err != nil {
    log.Printf("Error: %v", err)
}
fmt.Printf("%v\n", response)
```

**Symbols Response**

```go
map[data:map[brexchange:NSE brsymbol:RELIANCE-EQ exchange:NSE expiry: id:986 instrumenttype:EQ lotsize:1 name:RELIANCE strike:-1 symbol:RELIANCE tick_size:0.1 token:2885] status:success]
```

#### Search Example

```go
response, err := client.Search("NIFTY 25000 JUL CE", "NFO")
if err != nil {
    log.Printf("Error: %v", err)
}
fmt.Printf("%v\n", response)
```

**Search Response**

```go
map[data:[map[brexchange:NFO brsymbol:NIFTY28OCT25C25000 exchange:NFO expiry:28-OCT-25 instrumenttype:CE lotsize:75 name:NIFTY strike:25000 symbol:NIFTY28OCT2525000CE tick_size:0.05 token:58909] map[brexchange:NFO brsymbol:FINNIFTY28OCT25C25000 exchange:NFO expiry:28-OCT-25 instrumenttype:CE lotsize:65 name:FINNIFTY strike:25000 symbol:FINNIFTY28OCT2525000CE tick_size:0.05 token:57065] map[brexchange:NFO brsymbol:NIFTY20OCT25C25000 exchange:NFO expiry:20-OCT-25 instrumenttype:CE lotsize:75 name:NIFTY strike:25000 symbol:NIFTY20OCT2525000CE tick_size:0.05 token:45248] map[brexchange:NFO brsymbol:NIFTY14OCT25C25000 exchange:NFO expiry:14-OCT-25 instrumenttype:CE lotsize:75 name:NIFTY strike:25000 symbol:NIFTY14OCT2525000CE tick_size:0.05 token:42673] map[brexchange:NFO brsymbol:NIFTY07OCT25C25000 exchange:NFO expiry:07-OCT-25 instrumenttype:CE 
```

#### Expiry Example

```go
response, err := client.Expiry(
    "NIFTY",   // symbol
    "NFO",     // exchange
    "options", // instrument_type
)
if err != nil {
    log.Printf("Error: %v", err)
}
fmt.Printf("%v\n", response)
```

**Expiry Response**

```go
map[data:[30-SEP-25 07-OCT-25 14-OCT-25 20-OCT-25 28-OCT-25 04-NOV-25 25-NOV-25 30-DEC-25 31-MAR-26 30-JUN-26 29-DEC-26 29-JUN-27 28-DEC-27 27-JUN-28 26-DEC-28 26-JUN-29 24-DEC-29 25-JUN-30] message:Found 18 expiry dates for NIFTY options in NFO status:success]
```

#### Funds Example

```go
response, err := client.Funds()
if err != nil {
    log.Printf("Error: %v", err)
}
fmt.Printf("%v\n", response)
```

**Funds Response**

```json
{
  "data": {
    "availablecash": "93.81",
    "collateral": "0.00",
    "m2mrealized": "-0.13",
    "m2munrealized": "0.00",
    "utiliseddebits": "86.81"
  },
  "status": "success"
}
```

#### OrderBook Example

```go
response, err := client.OrderBook()
if err != nil {
    log.Printf("Error: %v", err)
}
fmt.Printf("%v\n", response)
```

```json
{
  "data": {
    "orders": [
      {
        "action": "BUY",
        "exchange": "NSE",
        "order_status": "canceled",
        "orderid": "25092400516306",
        "price": "20.30",
        "pricetype": "LIMIT",
        "product": "MIS",
        "quantity": "1",
        "symbol": "YESBANK",
        "timestamp": "14:28:56 24-09-2025",
        "trigger_price": 0
      },
      {
        "action": "BUY",
        "exchange": "NSE",
        "order_status": "complete",
        "orderid": "25092400496407",
        "price": 0,
        "pricetype": "MARKET",
        "product": "CNC",
        "quantity": "1",
        "symbol": "NHPC",
        "timestamp": "14:14:26 24-09-2025",
        "trigger_price": 0
      },
      {
        "action": "BUY",
        "exchange": "NSE",
        "order_status": "complete",
        "orderid": "25092400496406",
        "price": 0,
        "pricetype": "MARKET",
        "product": "MIS",
        "quantity": "1",
        "symbol": "YESBANK",
        "timestamp": "14:14:26 24-09-2025",
        "trigger_price": 0
      },
      {
        "action": "SELL",
        "exchange": "NSE",
        "order_status": "complete",
        "orderid": "25092400492663",
        "price": 0,
        "pricetype": "MARKET",
        "product": "MIS",
        "quantity": "2",
        "symbol": "YESBANK",
        "timestamp": "14:11:10 24-09-2025",
        "trigger_price": 0
      },
    ],
    "statistics": {
      "total_buy_orders": 2,
      "total_completed_orders": 3,
      "total_open_orders": 0,
      "total_rejected_orders": 0,
      "total_sell_orders": 1
    }
  },
  "status": "success"
}
```

#### TradeBook Example

```go
response, err := client.TradeBook()
if err != nil {
    log.Printf("Error: %v", err)
}
fmt.Printf("%v\n", response)
```

**TradeBook Response**

```json
{
  "data": [
    {
      "action": "BUY",
      "average_price": "21.30",
      "exchange": "NSE",
      "orderid": "25092400504502",
      "product": "MIS",
      "quantity": "3",
      "symbol": "YESBANK",
      "timestamp": "14:20:38",
      "trade_value": 63.9
    },
    {
      "action": "SELL",
      "average_price": "21.28",
      "exchange": "NSE",
      "orderid": "25092400502096",
      "product": "MIS",
      "quantity": "2",
      "symbol": "YESBANK",
      "timestamp": "14:18:50",
      "trade_value": 42.56
    },
    {
      "action": "SELL",
      "average_price": "21.28",
      "exchange": "NSE",
      "orderid": "25092400502095",
      "product": "MIS",
      "quantity": "2",
      "symbol": "YESBANK",
      "timestamp": "14:18:50",
      "trade_value": 42.56
    },
    {
      "action": "BUY",
      "average_price": "86.40",
      "exchange": "NSE",
      "orderid": "25092400496407",
      "product": "CNC",
      "quantity": "1",
      "symbol": "NHPC",
      "timestamp": "14:14:26",
      "trade_value": 86.4
    },
    {
      "action": "BUY",
      "average_price": "21.29",
      "exchange": "NSE",
      "orderid": "25092400496406",
      "product": "MIS",
      "quantity": "1",
      "symbol": "YESBANK",
      "timestamp": "14:14:26",
      "trade_value": 21.29
    },
    {
      "action": "SELL",
      "average_price": "21.29",
      "exchange": "NSE",
      "orderid": "25092400492663",
      "product": "MIS",
      "quantity": "2",
      "symbol": "YESBANK",
      "timestamp": "14:11:10",
      "trade_value": 42.58
    },
    {
      "action": "BUY",
      "average_price": "21.32",
      "exchange": "NSE",
      "orderid": "25092400482144",
      "product": "MIS",
      "quantity": "1",
      "symbol": "YESBANK",
      "timestamp": "14:03:23",
      "trade_value": 21.32
    },
    {
      "action": "BUY",
      "average_price": "21.32",
      "exchange": "NSE",
      "orderid": "25092400475638",
      "product": "MIS",
      "quantity": "1",
      "symbol": "YESBANK",
      "timestamp": "13:58:19",
      "trade_value": 21.32
    }
  ],
  "status": "success"
}
```

#### PositionBook Example

```go
response, err := client.PositionBook()
if err != nil {
    log.Printf("Error: %v", err)
}
fmt.Printf("%v\n", response)
```

**PositionBook Response**

```json
{
  "data": [
    {
      "average_price": 0,
      "exchange": "NSE",
      "ltp": 0,
      "pnl": 0,
      "product": "MIS",
      "quantity": 0,
      "symbol": "YESBANK"
    },
    {
      "average_price": 86.4,
      "exchange": "NSE",
      "ltp": 86.29,
      "pnl": -0.11,
      "product": "CNC",
      "quantity": 1,
      "symbol": "NHPC"
    }
  ],
  "status": "success"
}
```

#### Holdings Example

```go
response, err := client.Holdings()
if err != nil {
    log.Printf("Error: %v", err)
}
fmt.Printf("%v\n", response)
```

**Holdings Response**

```json
{
  "data": {
    "holdings": [
      {
        "exchange": "NSE",
        "pnl": 0,
        "pnlpercent": 0,
        "product": "CNC",
        "quantity": 1,
        "symbol": "BHEL"
      },
      {
        "exchange": "NSE",
        "pnl": 0,
        "pnlpercent": 0,
        "product": "CNC",
        "quantity": 1,
        "symbol": "YESBANK"
      }
    ],
    "statistics": {
      "totalholdingvalue": 271.55,
      "totalinvvalue": 271.55,
      "totalpnlpercentage": 0,
      "totalprofitandloss": 0
    }
  },
  "status": "success"
}
```

#### Analyzer Status Example

```go
response, err := client.AnalyzerStatus()
if err != nil {
    log.Printf("Error: %v", err)
}
fmt.Printf("%v\n", response)
```

**Analyzer Status Response**

```json
{
  "data": {
    "analyze_mode": true,
    "mode": "analyze",
    "total_logs": 1
  },
  "status": "success"
}
```

#### Analyzer Toggle Example

```go
// Switch to analyze mode (simulated responses)
response, err := client.AnalyzerToggle(true)
if err != nil {
    log.Printf("Error: %v", err)
}
fmt.Printf("%v\n", response)
```

**Analyzer Toggle Response**

```json
{
  "data": {
    "analyze_mode": true,
    "message": "Analyzer mode switched to analyze",
    "mode": "analyze",
    "total_logs": 1
  },
  "status": "success"
}
```

#### LTP Data (Streaming Websocket)

```go
package main

import (
    "fmt"
    "log"
    "time"
    "github.com/marketcalls/openalgo-go/openalgo"
)

func main() {
    // Initialize OpenAlgo client
    client := openalgo.NewClient(
        "your_api_key",           // Replace with your actual OpenAlgo API key
        "http://127.0.0.1:5000",  // REST API host
        "v1",                     // API version
        "ws://127.0.0.1:8765",    // WebSocket host (optional)
    )

    // Connect to WebSocket
    if err := client.Connect(); err != nil {
        log.Fatalf("Failed to connect: %v", err)
    }
    defer client.Disconnect()

    // Define instruments to subscribe for LTP
    instruments := []openalgo.Instrument{
        {Exchange: "NSE", Symbol: "RELIANCE"},
        {Exchange: "NSE", Symbol: "INFY"},
    }

    // Callback function for LTP updates
    onLTP := func(data interface{}) {
        fmt.Println("LTP Update Received:")
        fmt.Println(data)
    }

    // Subscribe
    if err := client.SubscribeLTP(instruments, onLTP); err != nil {
        log.Printf("Error subscribing to LTP: %v", err)
    }

    // Run for a few seconds to receive data
    time.Sleep(10 * time.Second)

    // Unsubscribe
    client.UnsubscribeLTP(instruments)
}
```

#### Quotes (Streaming Websocket)

```go
package main

import (
    "fmt"
    "log"
    "time"
    "github.com/marketcalls/openalgo-go/openalgo"
)

func main() {
    // Initialize OpenAlgo client
    client := openalgo.NewClient(
        "your_api_key",           // Replace with your actual OpenAlgo API key
        "http://127.0.0.1:5000",  // REST API host
        "v1",                     // API version
        "ws://127.0.0.1:8765",    // WebSocket host (optional)
    )

    // Connect
    if err := client.Connect(); err != nil {
        log.Fatalf("Failed to connect: %v", err)
    }
    defer client.Disconnect()

    // Instruments list
    instruments := []openalgo.Instrument{
        {Exchange: "NSE", Symbol: "RELIANCE"},
        {Exchange: "NSE", Symbol: "INFY"},
    }

    // Callback for Quote updates
    onQuote := func(data interface{}) {
        fmt.Println("Quote Update Received:")
        fmt.Println(data)
    }

    // Subscribe to quote stream
    if err := client.SubscribeQuote(instruments, onQuote); err != nil {
        log.Printf("Error subscribing to quotes: %v", err)
    }

    // Keep the script running to receive data
    time.Sleep(10 * time.Second)

    // Unsubscribe
    client.UnsubscribeQuote(instruments)
}
```

#### Depth (Streaming Websocket)

```go
package main

import (
    "fmt"
    "log"
    "time"
    "github.com/marketcalls/openalgo-go/openalgo"
)

func main() {
    // Initialize OpenAlgo client
    client := openalgo.NewClient(
        "your_api_key",           // Replace with your actual OpenAlgo API key
        "http://127.0.0.1:5000",  // REST API host
        "v1",                     // API version
        "ws://127.0.0.1:8765",    // WebSocket host (optional)
    )

    // Connect
    if err := client.Connect(); err != nil {
        log.Fatalf("Failed to connect: %v", err)
    }
    defer client.Disconnect()

    // Instruments list for depth
    instruments := []openalgo.Instrument{
        {Exchange: "NSE", Symbol: "RELIANCE"},
        {Exchange: "NSE", Symbol: "INFY"},
    }

    // Callback for market depth updates
    onDepth := func(data interface{}) {
        fmt.Println("Market Depth Update Received:")
        fmt.Println(data)
    }

    // Subscribe to depth stream
    if err := client.SubscribeDepth(instruments, onDepth); err != nil {
        log.Printf("Error subscribing to depth: %v", err)
    }

    // Run for a few seconds to collect data
    time.Sleep(10 * time.Second)

    // Unsubscribe
    client.UnsubscribeDepth(instruments)
}
```
