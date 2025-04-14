# AngelOne

AngelOne, previously known as Angel Broking, is a prominent Indian stock brokerage firm offering a wide range of trading and investment services. For developers and fintech companies looking to integrate algorithmic trading capabilities, AngelOne provides an API gateway through its platform. This API, often referred to in the context of OpenAlgo integration, enables automated trading strategies, real-time market data access, and more.



The introductory phase of integrating with AngelOne's API typically involves:

**Enable TOTP**: Developers must register for an API key through AngelOne's developer portal, which serves as the authentication mechanism for API requests.

### **What is 2-factor authentication (2-FA)?** <a href="#what-is-2-factor-authentication-2-fa-3" id="what-is-2-factor-authentication-2-fa-3"></a>

The 2FA or 2-factor authentication is a user authentication process where brokers offer to traders/investors any two of the following three types of authentication mechanisms to users for logging in to their online trading account-

1. Knowledge factor (i.e. something that only the user knows) e.g. PIN, password, etc.
2. Possession factor (i.e. something that only the user has) e.g. OTP, security token, authenticator apps on smartphones, etc.
3. Biometrics (i.e. biological identity markers of the person) e.g. fingerprint, face id, pattern, etc.

### Manual TOTP Generation Process using Google Authenticator

Step 1 – Visit [smartapi.angelbroking.com/enable-totp](http://smartapi.angelbroking.com/enable-totp)

\
Step 2 – Enter your Angel One client id and trading terminal password **or MPIN**\
Step 3 – Enter the OTP sent to the Registered email & mobile. Once OTP is entered, you will see a QR code on the screen and a token number on the screen.

<figure><img src="../../.gitbook/assets/Angel TOTP.webp" alt=""><figcaption><p>Enable TOTP</p></figcaption></figure>

\
Step 4 – Open the **Google Authenticator App**. Install it from the Android Playstore or Apple Store\
Step 5 – Scan the QR code generated from our site on your authenticator app

Google authenticator now generates TOTP(Time-based OTP) and it is valid for 60 seconds. Use this for manual TOTP verification

<figure><img src="../../.gitbook/assets/Google-Authenticator.webp" alt=""><figcaption><p>Google Authenticator</p></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (83).png" alt=""><figcaption></figcaption></figure>



**API Key Registration**: Developers must register for an API key through AngelOne's developer portal, which serves as the authentication mechanism for API requests.

Goto the [Smart API Developer Portal](https://smartapi.angelbroking.com/signin) and login with your credentials. sign up if you are a new user.

Enter the App name, Redirect URL (use dummy url https://google.com) and your Angel Client ID

Save the generated apikey. Later we will be adding the apikey in the [environmental variable](https://docs.openalgo.in/getting-started/windows-installation/environmental-variables)

Here is a sample of how the details would appear in a .env file for reference:

```
BROKER_API_KEY = 'your_api_key_here'
BROKER_API_SECRET = 'your_api_secret_here'
REDIRECT_URL = 'http://127.0.0.1:5000/angel/callback'
```



Integrating OpenAlgo with AngelOne's API opens up possibilities for automated trading strategies, providing a powerful tool for traders and developers to exploit market opportunities efficiently. It's essential to follow best practices for API integration, including handling rate limits, managing API keys securely, and ensuring robust error handling and logging mechanisms are in place.
