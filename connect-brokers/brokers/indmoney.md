# IndMoney (INDstocks)

IndMoney (INDstocks) is a modern trading infrastructure platform offering APIs that allow traders and developers to build, automate, and execute trading strategies.\
Using IndMoney APIs, users can connect their IndMoney trading account with tools like **OpenAlgo** and create their own fully self-hosted, secure algo-trading setup.

### Steps for Integration

#### **Step 1:** Generate Your API Access Token

IndMoney uses a simple **Bearer Token** authentication model.

To generate a token:

Visit [**https://www.indstocks.com/app/api-trading**](https://www.indstocks.com/app/api-trading)

<figure><img src="../../.gitbook/assets/ind1.PNG" alt=""><figcaption></figcaption></figure>

1. Enter your registered mobile number and login with the help of OTP

<figure><img src="../../.gitbook/assets/ind2.PNG" alt=""><figcaption></figcaption></figure>

2. Open the **API Dashboard**

<figure><img src="../../.gitbook/assets/ind3.PNG" alt=""><figcaption></figcaption></figure>

3. Click **New Token** and generate token

<figure><img src="../../.gitbook/assets/ind5.PNG" alt=""><figcaption></figcaption></figure>

4. Copy the token shown on screen

This token will be used in OpenAlgo for all authenticated API requests.

**Note:**\
Tokens usually expire within **24 hours**. You must generate a fresh token when expired.

5. Setup static IP , by clicking small hexagon button next to New Token and save it.

<figure><img src="../../.gitbook/assets/ind4.PNG" alt=""><figcaption></figcaption></figure>
