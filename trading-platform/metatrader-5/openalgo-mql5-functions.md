# OpenAlgo MQL5 Functions

Below is the API documentation for the OpenAlgoAPI functions provided in your MQL5 header file:

***

## OpenAlgoAPI Function Documentation



### `PlaceOrder`

Places an order with the specified parameters by sending a POST request to the OpenAlgo API.

#### Parameters:

* `actionParam`: The trade action, either "BUY" or "SELL".
* `quantityParam`: The quantity of the asset to trade.
* `apiUrlParam`: The URL of the OpenAlgo API.
* `apiKeyParam`: The API key for authentication with OpenAlgo.
* `strategyParam`: The name of the trading strategy.
* `symbolParam`: The symbol for the asset being traded.
* `exchangeParam`: The exchange on which the trade will be executed.
* `productParam`: The product type of the order.
* `priceTypeParam`: The price type of the order.
* `priceParam` (optional): The price at which to place the order.
* `triggerPriceParam` (optional): The trigger price for stop-loss or stop-limit orders.
* `disclosedQuantityParam` (optional): The disclosed quantity of the order.

#### Usage:

```mql5
PlaceOrder("BUY", 10, "http://127.0.0.1:5000", "your_api_key", "Meta Strategy", "SBIN-EQ", NSE, MIS, MARKET);
```

### `PlaceSmartOrder`

Places an order that considers the current open position size and matches the position size given in the position book.

#### Parameters:

* Inherits all parameters from `PlaceOrder`.
* `positionSizeParam`: The size of the position to be matched for the order.

#### Usage:

```mql5
PlaceSmartOrder("SELL", 5, 10, "http://127.0.0.1:5000", "your_api_key", "ReversalStrategy", "TATASTEEL-EQ", NSE, MIS, LIMIT, 150.00);
```

### `ModifyOrder`

Modifies an existing order with new parameters.

#### Parameters:

* Inherits all parameters from `PlaceOrder`.
* `orderidParam`: The ID of the order to be modified.

#### Usage:

```mql5
ModifyOrder("123456789", "BUY", 10, 155.00, "http://127.0.0.1:5000", "your_api_key", "Meta Strategy", "YESBANK-EQ", NSE, CNC, LIMIT);
```

### `CancelOrder`

Cancels an existing order.

#### Parameters:

* `orderidParam`: The ID of the order to be canceled.
* `apiUrlParam`: The URL of the OpenAlgo API.
* `apiKeyParam`: The API key for authentication with OpenAlgo.
* `strategyParam`: The name of the trading strategy.

#### Usage:

```mql5
CancelOrder("123456789", "http://127.0.0.1:5000", "your_api_key", "Meta Strategy");
```

### `ClosePosition`

Closes all open positions associated with a given strategy.

#### Parameters:

* `apiUrlParam`: The URL of the OpenAlgo API.
* `apiKeyParam`: The API key for authentication with OpenAlgo.
* `strategyParam`: The name of the trading strategy to close positions for.

#### Usage:

```mql5
ClosePosition("http://127.0.0.1:5000", "your_api_key", "Meta Strategy");
```

### `CancelAllOrders`

Cancels all orders associated with a given strategy.

#### Parameters:

* `apiUrlParam`: The URL of the OpenAlgo API.
* `apiKeyParam`: The API key for authentication with OpenAlgo.
* `strategyParam`: The name of the trading strategy to cancel orders for.

#### Usage:

```mql5
CancelAllOrders("http://127.0.0.1:5000", "your_api_key", "Meta Strategy");
```

***

**Note**: Replace placeholder values like `"your_api_key"` and `"http://127.0.0.1:5000"` with actual API keys and URLs as required by your setup. All functions assume the existence of a network communication setup within the MetaTrader 5 environment, as well as an OpenAlgo API that responds to the specified endpoints. Ensure proper error handling and API rate limiting as per the service provider's specifications.
