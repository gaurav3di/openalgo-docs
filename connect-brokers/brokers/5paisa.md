# 5Paisa

OpenAlgo makes algorithmic trading accessible and straightforward by providing seamless integration with various brokers, including 5Paisa. This documentation will guide you through the process of configuring your 5Paisa account to work with OpenAlgo, ensuring you can automate your trading strategies efficiently. Follow the steps below to enable TOTP, generate APIs, and set up your environment for a smooth trading experience.

### **Step 1: Enable TOTP**

**Install Google Authenticator:**

* Download and install the Google Authenticator app on your phone.

**Set Up TOTP on 5Paisa:**

* **Login to 5Paisa Website:**
  * Navigate to the [5Paisa website](https://tradestation.5paisa.com/dashboard) and log in with your credentials.
* **Access TOTP Setup:**
  * Click on your login ID at the top right corner to bring out the menu.
  * Go to `Profile -> TOTP`.



<figure><img src="../../.gitbook/assets/image (38).png" alt=""><figcaption></figcaption></figure>

* **Enable TOTP:**
  * If you already have TOTP enabled but do not have the "TOTP Key," you will need to disable TOTP and then enable it again.
* **Enable External TOTP:**
  * Click on "Enable external TOTP."
  * Enter the OTP received on your phone/email.
  * Enter your 6-digit MPIN.
  * Scan the QR code in your Google Authenticator app.
  * Copy the TOTP KEY shown below the QR code and save it in a notepad for later use.
  * Enter the TOTP shown in the Google Authenticator app.
  * Click the "Enable" button.
* **Confirmation:**
  * You will see a message saying, "TOTP successfully enabled."

### **Step 2: Generate APIs**

1. **Login to 5Paisa Website:**
   * Navigate to the 5Paisa website and log in with your credentials.
2. **Access API Key Generation:**
   * Click on the menu on the top right next to your name.
   * Go to `Developer’s APIs -> Get API Keys`.
   * Keep this page open and proceed to the next step.

<figure><img src="../../.gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

### **Step 3: Save API Details**

1. **Save API Details:**
   * Save the generated User Key, User ID. Encryption Key, Later, you will add these details to the environmental variables along with the redirect URL.

<figure><img src="../../.gitbook/assets/image (40).png" alt=""><figcaption></figcaption></figure>

**Get the Client ID**&#x20;

**Login to 5Paisa Website:**

* Navigate to the [5Paisa website](https://tradestation.5paisa.com/dashboard) and log in with your credentials.

**Access MyProfile**

* Click on your login ID at the top right corner to bring out the menu.
* Go to `Profile -> MyProfile and get the`` `**`Client ID`**` ``as shown below`

\


<figure><img src="../../.gitbook/assets/image (41).png" alt=""><figcaption></figcaption></figure>

Save the generated User\_Key, User\_ID, client\_id and Encryption\_Key. Later we will be adding the apikey in the [environmental variable](https://docs.openalgo.in/getting-started/windows-installation/environmental-variables) along with the redirect url

Here is a sample of how the details would appear in a .env file for reference:

**Sample .env File:** Here is a sample of how the details should appear in a `.env` file for reference:

```
BROKER_API_KEY = 'User_Key:::User_ID:::client_id'
BROKER_API_SECRET = 'Encryption_Key'
REDIRECT_URL = 'http://127.0.0.1:5000/fivepaisa/callback'
```

#### Conclusion

By following the steps outlined in this guide, you have successfully configured your 5Paisa account for use with OpenAlgo. You can now leverage the power of algorithmic trading to enhance your trading strategies and make data-driven decisions. Should you encounter any issues or need further assistance, please refer to the OpenAlgo community or support resources. Happy trading!
