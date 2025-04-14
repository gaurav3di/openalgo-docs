# NodeJS

To install the OpenAlgo Node.js library, use npm:

```bash
npm install openalgo
```

## OpenAlgo Node.js Library Documentation

### Get the OpenAlgo API Key

Make sure that your OpenAlgo Application is running. Login to the OpenAlgo Application with valid credentials and get the OpenAlgo API key.

\
For detailed function parameters, refer to the [API Documentation](https://docs.openalgo.in/api-documentation/v1).

### Getting Started with OpenAlgo

First, import the OpenAlgo class from the library and initialize it with your API key:

```javascript
import OpenAlgo from 'openalgo';

// Replace 'YOUR_API_KEY' with your actual API key
// Specify the host URL with your hosted domain or ngrok domain.
// If running locally in Windows then use the default host value.
const openalgo = new OpenAlgo('YOUR_API_KEY', 'http://127.0.0.1:5000');
```

### Check OpenAlgo Version

```javascript
import { version } from 'openalgo';
console.log(version);
```

### PlaceOrder Example

To place a new order:

```javascript
const response = await openalgo.placeOrder({
    strategy: "NodeJS",
    symbol: "RELIANCE", 
    action: "BUY",
    exchange: "NSE",
    pricetype: "MARKET",
    product: "MIS",
    quantity: 1
});
console.log(response);
```

### PlaceSmartOrder Example

To place a smart order considering the current position size:

```javascript
const response = await openalgo.placeSmartOrder({
    strategy: "NodeJS",
    symbol: "TATAMOTORS",
    action: "SELL",
    exchange: "NSE",
    pricetype: "MARKET",
    product: "MIS",
    quantity: 1,
    positionSize: 5
});
console.log(response);
```

### BasketOrder Example

To place a new basket order:

```javascript
const basketOrders = [
    {
        symbol: "RELIANCE",
        exchange: "NSE",
        action: "BUY",
        quantity: 1,
        pricetype: "MARKET",
        product: "MIS"
    },
    {
        symbol: "INFY",
        exchange: "NSE",
        action: "SELL",
        quantity: 1,
        pricetype: "MARKET",
        product: "MIS"
    }
];

const response = await openalgo.basketOrder({
    strategy: "NodeJS",
    orders: basketOrders
});
console.log(response);
```

### SplitOrder Example

To place a new split order:

```javascript
const response = await openalgo.splitOrder({
    symbol: "YESBANK",
    exchange: "NSE",
    action: "SELL",
    quantity: 105,
    splitSize: 20,
    pricetype: "MARKET",
    product: "MIS",
    strategy: "NodeJS"
});
console.log(response);
```

### ModifyOrder Example

To modify an existing order:

```javascript
const response = await openalgo.modifyOrder({
    orderId: "123456789",
    strategy: "NodeJS",
    symbol: "INFY",
    action: "SELL",
    exchange: "NSE",
    pricetype: "LIMIT",
    product: "CNC",
    quantity: 2,
    price: 1900
});
console.log(response);
```

### CancelOrder Example

To cancel an existing order:

```javascript
const response = await openalgo.cancelOrder({
    orderId: "123456789",
    strategy: "NodeJS"
});
console.log(response);
```

### CancelAllOrder Example

To cancel all open orders and trigger pending orders:

```javascript
const response = await openalgo.cancelAllOrder({
    strategy: "NodeJS"
});
console.log(response);
```

### ClosePosition Example

To close all open positions across various exchanges:

```javascript
const response = await openalgo.closePosition({
    strategy: "NodeJS"
});
console.log(response);
```

### OrderStatus Example

To get the current order status:

```javascript
const response = await openalgo.orderStatus({
    orderId: "123456789",
    strategy: "NodeJS"
});
console.log(response);
```

### OpenPosition Example

To get the current open position:

```javascript
const response = await openalgo.openPosition({
    strategy: "NodeJS",
    symbol: "YESBANK",
    exchange: "NSE",
    product: "CNC"
});
console.log(response);
```

### Quotes Example

To get real-time quotes:

```javascript
const response = await openalgo.quotes({
    symbol: "RELIANCE", 
    exchange: "NSE"
});
console.log(response);
```

### Depth Example

To get market depth data:

```javascript
const response = await openalgo.depth({
    symbol: "SBIN", 
    exchange: "NSE"
});
console.log(response);
```

### History Example

To get historical data:

```javascript
const response = await openalgo.history({
    symbol: "SBIN",
    exchange: "NSE",
    interval: "5m",
    startDate: "2025-01-01",
    endDate: "2025-01-05"
});
console.log(response);
```

### Intervals Example

To get supported time intervals:

```javascript
const response = await openalgo.intervals();
console.log(response);
```

### Symbol Example

To get symbol information:

```javascript
const response = await openalgo.symbol({
    symbol: "RELIANCE",
    exchange: "NSE"
});
console.log(response);
```

### Funds Example

To get account funds information:

```javascript
const response = await openalgo.funds();
console.log(response);
```

### OrderBook Example

To get the order book:

```javascript
const response = await openalgo.orderbook();
console.log(response);
```

### TradeBook Example

To get the trade book:

```javascript
const response = await openalgo.tradebook();
console.log(response);
```

### PositionBook Example

To get the position book:

```javascript
const response = await openalgo.positionbook();
console.log(response);
```

### Holdings Example

To get holdings:

```javascript
const response = await openalgo.holdings();
console.log(response);
```

Please refer to the documentation and consult the API reference for details on optional parameters:

* [API Documentation](https://docs.openalgo.in/api-documentation/v1)
* [Order Constants](https://docs.openalgo.in/api-documentation/v1/order-constants)
