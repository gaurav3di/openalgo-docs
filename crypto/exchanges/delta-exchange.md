# Delta Exchange

OpenAlgo's `deltaexchange` plugin is the repository's crypto-derivatives adapter. Its `plugin.json` declares broker type `crypto`, exchange `CRYPTO`, and leverage configuration support. The adapter maps supported Delta Exchange order, account, data, and streaming operations into OpenAlgo's normalized interfaces.

Product availability, fees, settlement, API-key requirements, and account eligibility are controlled by Delta Exchange and can change. OpenAlgo does not provide legal or tax classification; verify current exchange documentation and the rules that apply to your location before trading.

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

The Delta Exchange plugin supports automated crypto-derivatives workflows where the connected account and instrument are available. Protect API credentials, use the exchange's current network controls, handle API failures explicitly, and test with the exchange's test environment before live use. Users remain responsible for verifying the legal, tax, and account rules that apply to their location and activity.
