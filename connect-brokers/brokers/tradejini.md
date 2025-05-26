# Tradejini

TradeJini

TradeJini is a progressive brokerage firm known for its reliable execution, low-cost trading, and developer-friendly CubePlus API platform. Designed to support modern algo trading systems, TradeJini provides a robust infrastructure for both retail and institutional traders. Whether you're an individual trader or building advanced trading automation, TradeJini offers the right tools to seamlessly integrate market data and trading functionalities.

***

### Steps for Integration:

#### 1. Log in to the TradeJini Developer Portal

Start by accessing the CubePlus Developer Portal at\
https://api.tradejini.com/developer-portal/main

<figure><img src="../../.gitbook/assets/1 (1).JPG" alt=""><figcaption></figcaption></figure>

* Click on **Login**.
* You can log in using either:
  * Your **email address and password**, or
  * By clicking the **TradeJini** logo for broker login.

> If you don’t have an account yet, click **SignUp** to register.
>
> ![](../../.gitbook/assets/4.JPG)

***

#### 2. Create a New App

After logging in:

* Navigate to the **Apps** section.
* Under **Individual Access**, click **Create New App**.
* Click **Submit** to register your app.
* An **API Key** will be created for authenticated API usage



<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

***

#### 3. Retrieve API Credentials

After generating the API Key:

* **Client Code** will be your `api_key`.
* **API Key** from the app will serve as your `api_secret`.

<figure><img src="../../.gitbook/assets/3.JPG" alt=""><figcaption></figcaption></figure>

***

### Configuration:

Here is how you would typically set up your environment variables in a `.env` file for TradeJini:

```bash
bashCopyEditBROKER_API_KEY = 'your_tradejini_clientcode_here'
BROKER_API_SECRET = 'your_tradejini_apikey_here'
REDIRECT_URL = 'http://127.0.0.1:5000/tradejini/callback'
```

These credentials will be used by OpenAlgo to authenticate, fetch session tokens, and access market feeds or place orders.

***

### Integration Benefits

Integrating with the TradeJini API via OpenAlgo opens up a world of possibilities for algorithmic trading and real-time data access. The CubePlus API offers:

* High-performance WebSocket feed for LTP, OHLC, depth, and indices.
* REST API endpoints for order management and historical data.
* Support for complex order types and multi-segment instruments.

To ensure a reliable experience with TradeJini’s API:

* Handle rate limits and WebSocket throttling appropriately.
* Store and manage tokens securely.
* Implement robust error handling, retry mechanisms, and logging.

By following best practices, developers and traders can leverage the TradeJini infrastructure to build high-frequency, data-driven strategies with confidence.
