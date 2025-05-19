# Dhan(Sandbox)

**What is Dhan SandBox API?**

Dhan SandBox API is a simulated testing environment designed for developers and traders to experiment with Dhan’s trading and market data APIs without needing a live brokerage account or actual capital. It replicates the real trading environment but ensures zero risk by not executing trades on the actual exchange. It is ideal for testing integrations, validating order logic, and experimenting with strategies.

***

**Who Should Integrate Dhan SandBox API with OpenAlgo?**

OpenAlgo is an open-source algo trading automation framework that connects with platforms like TradingView, Amibroker, and custom dashboards. Integrating Dhan’s SandBox API with OpenAlgo is highly recommended for:

* **Algo Developers** building broker connectors.
* **Traders** who want to simulate real-time automated strategies before going live.
* **Educators and Trainers** who want a safe demo environment.
* **Early-stage Fintech Builders** exploring trading APIs.
* **QA and Support Teams** needing to replicate live bugs or behaviors in a controlled setup.

***

**Step-by-Step: How to Get Started with Dhan SandBox API**

Here’s how to register, access tokens, and start testing with OpenAlgo:

#### 1. **Visit DhanHQ DevPortal**

You’ll land on a login screen where you can enter your email or mobile number.

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

***

#### 2. **Register (if new)**

If you’re a new user, click to register. Provide:

* Email ID
* Mobile Number
* Your Name

Proceed to the next step.

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

***

#### 3. **OTP Verification**

You will receive two OTPs:

* One on your **email**
* One on your **mobile**

Enter both to complete verification.

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

***

#### 4. **Access Your Sandbox Token and Client ID**

Once inside, go to the **Sandbox** tab. You’ll see:

* Application Name (e.g., `openalgo`)
* **Access Token** (JWT for authorization)
* **Client ID** (unique to your application)
* API Playground and documentation links

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

These credentials are what you will use inside OpenAlgo’s configuration or `.env` file to test trading orders programmatically.

***

#### Retrieve API Credentials

Once you've successfully logged into the DhanHQ DevPortal and created your sandbox application:

1. **Copy the Client ID**
   * This is your `BROKER_API_KEY`
   * It uniquely identifies your application (e.g., `2505178942`)
2. **Copy the Access Token**
   * This is your `BROKER_API_SECRET`

It acts like a password (JWT format) to authenticate your API requests.

#### Configuration for OpenAlgo

Inside your OpenAlgo project (or any backend using Dhan APIs), create or update the `.env` file with the following keys:

```bash
BROKER_API_KEY = 'your_dhan_sandbox_clientid_here' 
BROKER_API_SECRET = 'your_dhan_sandbox_token_here' 
REDIRECT_URL = 'http://127.0.0.1:5000/dhan_sandbox/callback'
```

**Summary: Why It Matters for OpenAlgo**

Integrating Dhan SandBox with OpenAlgo means:

* Safer testing for order placement, cancellation, and position handling.
* No compliance delay since no KYC is needed.
* Faster debugging and integration turnaround.
* Enables workshops, testing, and community support for algo development.

By setting up your sandbox environment with Dhan and OpenAlgo, you accelerate your readiness for live algorithmic trading while ensuring a risk-free development experience.
