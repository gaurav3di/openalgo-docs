# Tradingview

TradingView is a popular charting and analysis platform used by traders for market analysis, and it also offers features that support automated trading through the use of webhooks. Automated trading works.

#### What is a Webhook?

A webhook is a method for an app to provide other applications with real-time information. A webhook delivers data to other applications as it happens, meaning you get data immediately. In the context of TradingView, webhooks are used to send signals or alerts to external systems (like a brokerage or a trading bot) whenever certain predefined conditions are met.



{% embed url="https://www.youtube.com/watch?v=R7n08unEKeI" %}

#### Setting up Automated Trading using TradingView Webhooks

1. **Define Trading Strategy**: First, you create a trading strategy in TradingView using its Pine Script programming language. This strategy could be based on technical analysis indicators, patterns, or other criteria.
2. **Configure Alerts**: Once your strategy is defined, you set up alerts in TradingView that trigger when specific conditions of your strategy are met. These conditions could be anything that your strategy considers a signal, such as the crossing of moving averages or reaching a certain RSI level.
3. **Set Up Webhook URL**: In the alert settings, you specify a webhook URL to which TradingView will send POST requests when the alert conditions are triggered. This URL is typically provided by the external service or trading bot that will execute the trades on your behalf. It's essential to ensure that this service can accept and process TradingView's webhook notifications.

**OpenAlgo Configuration**



> Ensure that if you are a windows user ngrok host url  is setup and configured in the .env file. If you are linux user ensure that host name (domain name/ip address) is configured in the .env file

Login to the OpenAlgo Application and go to the tab **Tradingview**. The page comes with **Tradingview JSON Generator.**

Enter the valid trading **symbol**, select the **product** type, and press the **Generate JSON** Button

<figure><img src="../.gitbook/assets/image (27).png" alt=""><figcaption></figcaption></figure>

Copy the Webhook URL and the Alert Message for a Tradingview Strategy and configure the same in your tradingview strategy

<figure><img src="../.gitbook/assets/image (28).png" alt=""><figcaption></figcaption></figure>

Enter the Webhook URL

<figure><img src="../.gitbook/assets/image (29).png" alt=""><figcaption></figcaption></figure>
