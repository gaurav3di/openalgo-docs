# ICICI Direct

ICICIDirect is a premier trading platform in India offering a comprehensive suite of investment products, including equity, derivatives, mutual funds, and more. The ICICIDirect Breeze API allows developers to build and integrate trading applications, automate trading strategies, and access market data seamlessly.

In this guide, we will walk you through the steps to obtain your API credentials, including the API Key and API Secret, necessary for integrating ICICIDirect Breeze API with OpenAlgo.

#### API Login Procedure

**Access the ICICIDirect Breeze API Portal**:

* Visit the [ICICIDirect Breeze API Portal](https://api.icicidirect.com/apiuser/home).

<figure><img src="../../.gitbook/assets/image (22).png" alt=""><figcaption></figcaption></figure>

**Login**:

* Click on the Login button.
* Enter your ICICI Direct Login ID and Password.
* Enter the OTP received on your registered mobile number.

<figure><img src="../../.gitbook/assets/image (23).png" alt=""><figcaption></figcaption></figure>

#### Getting API Key and API Secret

**Register an App**:

* After logging in, navigate to the "Register an App" section.
* Enter the following details:
  * **App Name**: `openalgo`
  * **Redirect URL**: `http://127.0.0.1:5000/icici/callback`

<figure><img src="../../.gitbook/assets/image (24).png" alt=""><figcaption></figcaption></figure>

**Generate API Credentials**:

* Once you have registered your app, the API Key and API Secret will be generated.
* Copy the API Key and API Secret.

<figure><img src="../../.gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

#### Sample .env File

Create a `.env` file in your project directory and add the following lines. Replace `your_api_key_here` and `your_api_secret_here` with the actual values you copied earlier:

```
BROKER_API_KEY = 'your_api_key_here'
BROKER_API_SECRET = 'your_api_secret_here'
REDIRECT_URL = 'http://127.0.0.1:5000/icici/callback'
```

By following the steps outlined in this guide, you will have successfully obtained your ICICIDirect Breeze API credentials and configured them for use with OpenAlgo. This integration allows you to leverage the powerful trading and market data features provided by ICICIDirect to enhance your trading strategies and applications.

For further assistance, refer to the [ICICIDirect Breeze API documentation](https://api.icicidirect.com/apiuser/home) or contact their support team.
