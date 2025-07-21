# Groww

Groww is currently the #1 stockbroker in India by active clients, offering a seamless digital investing experience to millions of retail investors. With its recent launch of trading APIs, Groww has opened the door for developers and fintech platforms to integrate custom trading workflows. In the context of OpenAlgo integration, Groww’s trading APIs provide capabilities for automated order placement, real-time market data access, and custom strategy execution for retail traders.

### How to Get Groww API Key and Secret?

<figure><img src="../../.gitbook/assets/image (109).png" alt=""><figcaption></figcaption></figure>

To integrate OpenAlgo with Groww, follow the steps below to generate your API key (access token):

1. Visit the [Groww Trade API Portal](https://groww.in/trade-api) and click on **Getting Started**.
2. Login using your Groww account credentials.
3. Click the **Generate API Token** button to create APIKey and APISecret

<figure><img src="../../.gitbook/assets/Screenshot 2025-07-21 at 5.16.32 PM.png" alt=""><figcaption></figcaption></figure>

Click on the **Generate API Keys** button and Select API Key and Secret an Enter the Token Name as "OpenAlgo" or any name of your choice and press **Generate API Key** button

<figure><img src="../../.gitbook/assets/Screenshot 2025-07-21 at 5.14.42 PM.png" alt=""><figcaption></figcaption></figure>

### Copy the generated Grow API Key and API Secret

<figure><img src="../../.gitbook/assets/Screenshot 2025-07-21 at 5.15.17 PM.png" alt=""><figcaption></figcaption></figure>

### Important Points

* Do **not share** your access token with untrusted third-party platforms or individuals.

#### Configuration:

Here is how you would typically set up your environment variables in a .env file for Groww's API:

```bash
BROKER_API_KEY = 'your_groww_apikey_here'
BROKER_API_SECRET = 'your_groww_apisecret_here'
REDIRECT_URL = 'http://127.0.0.1:5000/groww/callback'
```

Integrating OpenAlgo with Groww's trading API allows retail traders to leverage India’s most popular stockbroking platform for building and deploying their own algorithmic trading strategies. It’s a powerful combination for automating trading decisions while using a broker trusted by millions. As with any integration, be sure to handle token management carefully, log errors effectively, and build retry mechanisms to handle token expiry and downtime.
