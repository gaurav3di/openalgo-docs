# Dhan

Dhan is an innovative financial services provider focused on leveraging technology to offer a dynamic trading and investment platform. Recognized for its intuitive interface and powerful trading capabilities, Dhan strives to enhance the trading experiences of its users. The platform serves a diverse range of customers, from novice investors to seasoned traders, providing tools that support various investment strategies and trading styles.



{% embed url="https://www.youtube.com/watch?v=EqCLLRUvtBg" %}

For developers and algo traders looking to use the Dhan API, the integration process generally involves the following steps:

#### Steps for Integration:

1. **Log into Dhan Web Portal**: Visit the Dhan Web Portal at [web.dhan.co](http://web.dhan.co/) to start the integration process.
2. **Navigate to Profile**: Click on "DhanHQ Trading API's" under your profile options to access the API .![](<../../.gitbook/assets/image (36).png>)
3.  **Generate API Token**:

    * Under the "Trading APIs" tab, click on "Generate token".
    * Enter "OpenAlgo" as the Application Name.
    * Set the Token Validity for 30 days.
    * Click on the "Generate Token" button to create your API token.

    <figure><img src="../../.gitbook/assets/image (37).png" alt=""><figcaption></figcaption></figure>
4. **Retrieve API Credentials**:
   * Copy the generated token id, which will serve as your `apisecret`.
   * Go back to your profile page by clicking "My Profile on Dhan".
   * Copy the Client ID, which will be your `apikey`.

#### Configuration:

Here is how you would typically set up your environment variables in a .env file for Dhan's API:

```
BROKER_API_KEY = 'your_dhan_clientid_here'
BROKER_API_SECRET = 'your_dhan_token_here'
REDIRECT_URL = 'http://127.0.0.1:5000/dhan/callback'
```

Integrating with the Dhan API opens up new possibilities for implementing automated and algorithmic trading strategies. It provides developers and traders with a robust platform to access market opportunities efficiently and effectively. To fully leverage the capabilities of the Dhan API, it is essential to follow best practices for API integration. This includes careful management of API rate limits, secure handling of API keys, and thorough error handling and logging. These practices help ensure a dependable and optimal trading experience using Dhan's advanced technological infrastructure.
