---
description: New Broker Integeration with OpenAlgo - Checklist
---

# Broker Integration Checklist

***

## ✅ OpenAlgo Broker Integration Checklist (Updated)

***

### 1. Pre-Integration Checks

* Review broker's API documentation (REST, WebSocket, SDKs).
* Confirm sandbox or test environment availability.
* Identify rate limits, session expiry behavior, reconnection strategies.

***

### 2. Authentication

* Verify token generation (API Key + Secret or OAuth flow).
* Ensure successful login redirects to dashboard or access token endpoint.
* Check token refresh or re-authentication mechanism.

***

### 3. Funds & Margin

Fetch and normalize:

* Available Cash
* Collateral
* Utilized Margin
* Realized & Unrealized P\&L

✔ Store in OpenAlgo’s format via `/broker/api/funds.py`

***

### 4. Master Contract Handling

* Download and persist contracts for NSE, NFO, BSE, BFO, MCX, CDS, INDEX.
* Normalize symbols into OpenAlgo’s format.
* Validate `NSE_INDEX` and `BSE_INDEX` symbols.
* Validate special characters in symbols (e.g. `M&M`).

📘 Refer: [Symbol Format Guide](https://docs.openalgo.in/symbol-format)

***

### 5. Order APIs

Transform broker fields into OpenAlgo common format.

#### Place Order

* Test all Order Types:
  * MARKET, LIMIT, SL, SL-MKT
* Verify across all supported exchanges:
  * NSE, BSE (Cash), NFO/BFO (F\&O), MCX (Commodities), Currencies
* Ensure accurate field mapping for symbol, order type, qty, price, product, etc.

#### Modify Order

* **NOTE:** OpenAlgo **does not support** order type conversion (e.g., Limit → Market).
* Only support modifying Limit orders to other Limit orders (change price, quantity).
* Block or log attempts to convert order types.

#### Cancel Order(s)

* Support single order cancellation.
* Support bulk cancellation across segments.

#### Smart Orders (PlaceSmartOrder)

* Test across all supported exchanges (NSE, NFO, MCX, CDS, BSE).
* Test **position size increase** → should trigger order for difference.
* Test **position size decrease** → square-off excess quantity.
* Test with `position_size = 0` → must square off full open position.
* If `position_size` is same as open position → no order should be triggered.
* Validate against actual values in PositionBook.

***

### 6. Order Status & Books

#### OrderBook

* Validate:
  * Trading Symbol, Exchange, Transaction Type
  * Price, Trigger Price, Quantity, Product Type
  * Order ID, Status, Timestamp
* Confirm Order Stats: Total Buy/Sell/Open/Rejected/Completed

#### TradeBook

* Ensure mapping of:
  * Transaction Type, Fill Price, Quantity, Order ID, Fill Time

***

### 7. Positions

#### PositionBook

* Map:
  * Net Qty, Avg Price, LTP, P\&L, Product Type



#### OpenPosition

* Fetch open position quantity from broker.
* Validate broker → OpenAlgo symbol transform.
* Test for all supported exchanges and indices (NSE\_INDEX, BSE\_INDEX).

***

### 8. Portfolio & Holdings

* Validate:
  * Trading Symbol, Quantity, Exchange, Product
  * Investment Value, Current Value, Absolute & % P\&L
* Confirm Portfolio stats:
  * Total Holding Value, Investment Value, Total P\&L

***

### 9. Market Data APIs

#### Quotes

* Test for all supported exchanges and indices (NSE\_INDEX, BSE\_INDEX).
* Validate special characters (e.g. `M&M`).
*   If REST Quote API is not available:

    * Fetch via broker WebSockets.



#### Depth (Market Ladder)

* Ensure L5 depth is mapped correctly.
*   If REST Depth API is not available:

    * Fetch via broker WebSockets.



#### Historical Data

* Test intervals: 1m, 3m, 5m, 15m, 1h, 1d, W, M.
* If intraday candles are missing:
  * Backfill with Quotes API.
* Chunk data if date range exceeds limits.
* Ensure timestamps are in **IST**.

***

### 10. Final Testing & Edge Cases

* End-to-end test: Place → Modify → Cancel → Square Off.  Perform complete order lifecycle test
* Test partial fills, expired sessions, rejected orders.
* Validate consistent error handling and fallback response.
* Run integration tests across all supported segments

***

### 🔒 Additional Recommended Checks

* Ensure secure use of `.env` for API keys.
* Avoid exceeding rate limits while integerating broker API



***

Let me know if you’d like a version of this adapted for onboarding documentation, automated test scripts, or as a GitHub Wiki entry.
