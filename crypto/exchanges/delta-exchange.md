# Delta Exchange

Delta Exchange is a global cryptocurrency derivatives exchange founded in 2018 by Pankaj Balani, Jitender Tokas, and Saurabh Goyal, offering futures, perpetual swaps, and options trading on Bitcoin, Ethereum, and leading altcoins. Registered with India's Financial Intelligence Unit (FIU-India) and fully compliant with AML and KYC regulations, Delta Exchange stands as the first platform in India to offer crypto options trading.

The platform specialises in cryptocurrency derivatives with INR settlement, allowing Indian traders to deposit and withdraw funds directly from their bank accounts without needing to hold any stablecoins or crypto tokens. Its derivatives trading model, settled entirely in INR, means trades are classified as speculative business income — exempting users from the 30% capital gains tax and 1% TDS that typically apply to direct crypto transactions in India.

Delta Exchange offers a diverse product suite covering futures with daily, weekly, and monthly expiries on BTC, ETH, XRP, XLM, AAVE, and BNB; perpetual swaps for indefinite position holding; and options on BTC and ETH — the only platform in India to offer this. The exchange also launched "Trackers" in August 2025 — innovative derivative contracts that mirror crypto price movements without requiring direct asset ownership, available at a trading fee of just 0.05% compared to roughly 0.5% on spot markets.

For developers and algo traders looking to integrate Delta Exchange into OpenAlgo, the platform provides a comprehensive REST API and WebSocket feed.

### API Key Registration

To integrate OpenAlgo with Delta Exchange India, you need to generate an API key from within your Delta Exchange account. The API key serves as the authentication mechanism for all programmatic trading requests.

#### Step 1 – Log in to Delta Exchange India

Visit [delta.exchange](https://delta.exchange) and log in to your account using your Gmail and password

<figure><img src="../../.gitbook/assets/Delta1 (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/Delta2 (1).png" alt=""><figcaption></figcaption></figure>

#### Step 2 – Navigate to AlgoHub → APIs

<figure><img src="../../.gitbook/assets/Delta3 (1).png" alt=""><figcaption></figcaption></figure>

Once logged in, click on the **AlgoHub** menu in the top navigation bar. From the dropdown, select **APIs** ("Create API key and Start Trading"). This takes you to the Delta Exchange API landing page where you can manage your API keys.

#### Step 3 – Create a New API Key

<figure><img src="../../.gitbook/assets/Delat5 (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/Delta6 (1).png" alt=""><figcaption></figcaption></figure>



Click the **Create API Key** button on the API page. You will be taken to the API key creation form at delta.exchange/app/account/manageapikeys . Fill in the following details:

* **Account Name** – Select **Main** (or the sub-account you want the key associated with)
* **API Key Name** – Enter a recognisable name, e.g., openalgo
* **Whitelisted IP** – Enter the IP address of your OpenAlgo server. Only whitelisted IPs will be permitted to interact with the trading API. Add the IP and select it from the **IPs to whitelist** dropdown.
* **Permissions** – Enable both:
  * **Read Data** – Enabled by default for all API keys
  * **Trading** – Must be explicitly selected to allow order placement

#### Step 4 – Save the API Key and Secret

After submitting the form, Delta Exchange will display your **API Key** and **API Secret**. Copy and store them securely — the secret will not be shown again. Add them to your OpenAlgo `.env` file as follows:

```
BROKER_API_KEY = 'your_api_key_here'
BROKER_API_SECRET = 'your_api_secret_here'
REDIRECT_URL = 'http://127.0.0.1:5000/deltaexchange/callback'
```

Integrating OpenAlgo with Delta Exchange opens up possibilities for automated crypto derivatives trading — futures, perpetuals, and options — within a fully compliant Indian regulatory framework. As with all broker integrations, it is essential to follow best practices including secure API key management, IP whitelisting, robust error handling, and thorough testing on the testnet environment before going live.
