# ICICI Direct

> **Disclaimer**
>
> OpenAlgo is an open-source algorithmic trading platform (https://www.openalgo.in/) designed to facilitate individual traders in automating their trading by providing open-source code that interacts with supported brokerage APIs, including ICICI Direct’s Breeze API. OpenAlgo is not affiliated with, endorsed by, or in any formal partnership with ICICI Securities Limited ("ICICI Direct"). The inclusion of ICICI Direct in our documentation indicates API compatibility only and does not imply any official agreement or association.
>
> OpenAlgo.in does not host the application or any trading infrastructure. Individual traders are responsible for downloading, deploying, and managing the open-source code on their own local machines. All user data, including personal or sensitive information such as login IDs, passwords, or OTPs, is hosted exclusively on the user’s machine, and OpenAlgo does not collect, store, or process any such data. All interactions with ICICI Direct’s Breeze API occur directly through ICICI Direct’s official portal, where users enter their credentials securely within ICICI Direct’s system.
>
> OpenAlgo does not provide trading strategies, signals, or investment advice. The platform is intended solely to enable individual users to develop and execute their own automated trading strategies using the open-source code, which is publicly available for audit (https://github.com/marketcalls/OpenAlgo). Users must comply with ICICI Direct’s terms of service, API usage policies, and all applicable laws, including the Digital Personal Data Protection Act, 2023.
>
> By using OpenAlgo to interact with ICICI Direct’s Breeze API, users acknowledge that they do so at their own risk and are solely responsible for managing their infrastructure, ensuring compliance with ICICI Direct’s policies, and addressing any risks associated with automated trading.



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
