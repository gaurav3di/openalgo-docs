# Windows Server Installation

#### Why Choose a Server over a Desktop?

* **Reliability and Stability:** Servers provide a stable and dedicated environment for your trading software. Unlike a desktop, servers are designed for constant operation and are less susceptible to disruptions like power outages, system crashes, or internet issues.
* **Scalability:** If your trading algorithms become more complex or require access to vast amounts of data, you can easily scale up your server resources to accommodate those needs.
* **Security:** Servers offer enhanced security measures to protect your trading strategies and data. You can control access, enforce strict firewall rules, and implement encryption protocols for better data protection.
* **Accessibility:** You can access your server and trading platform remotely, regardless of your location. This allows for greater flexibility and control over your operations.

This video tutorial guides you through installing OpenAlgo and [Amibroker](https://docs.openalgo.in/trading-platform/amibroker) on a Windows Server via AWS, specifically targeting the Mumbai data center for optimal trading latency. It covers the complete setup process, from launching an AWS instance to configuring necessary software and testing the trading setup. For troubleshooting use [community support](https://docs.openalgo.in/community-support) for assistance.



{% embed url="https://www.youtube.com/watch?v=q3VD0hXO2Ps" %}

#### Step-by-Step Installation Process:

1. **Create an AWS Account:** If you don't have one already, sign up for an AWS account at [aws.amazon.com](https://aistudio.google.com/app/aws.amazon.com). AWS offers a free tier for new users, allowing you to try out the services without any initial cost.
2. **Launch an EC2 Instance:** Navigate to the EC2 service within the AWS console. Choose a Windows Server 2022 Base image for your instance and select a suitable instance type (for example t2.medium).
3. **Create a Key Pair:** This is crucial for securely connecting to your instance. Choose a unique key pair name (like "OpenAlgo-Server") and save the downloaded private key file in a secure location.
4. **Install Python:** Download the appropriate version of Python (for Amibroker compatibility, 3.11.9 is recommended) from [python.org](https://aistudio.google.com/app/python.org). During installation, ensure you select "Add python.exe to PATH" for easy access.
5. **Install VS Code:** Download VS Code from [code.visualstudio.com](https://aistudio.google.com/app/code.visualstudio.com). Once installed, go to the Extensions tab and search for "Python." Install the official Python extension to get all the required tools for coding.
6. **Install Git:** Download Git for Windows from [git-scm.com](https://aistudio.google.com/app/git-scm.com) and install it on your EC2 instance. Git is essential for downloading and managing OpenAlgo, which is hosted on GitHub.
7. **Installed Visual C++ Redistributable :** Download the [Visual C++ redistributable](https://learn.microsoft.com/en-us/cpp/windows/latest-supported-vc-redist?view=msvc-170#latest-supported-redistributable-version) as sometimes Windows Server comes with minimal installation. Packages like Numba which is one of the dependent of OpenAlgo Python SDK requires compilation in Run time.
8. **Download OpenAlgo:** Open your VS Code, open a new terminal window, and use the Git command to download OpenAlgo from GitHub:  This command will create a new folder named "OpenAlgo" containing the OpenAlgo codebase as shown in the video.
9. **Install Dependencies:** Navigate to the "OpenAlgo" folder using the cd command. Open the "requirements.txt" file. This file contains all the necessary dependencies for OpenAlgo. In the VS Code terminal, run the command: **pip install -r requirements.txt**. This will install all the required libraries for OpenAlgo to run smoothly.
10. **Obtain Broker API Credentials:** Log into your Broker and obtain the API key, API Secret key, and the redirect URL (could vary broker to broker. Hence,it is recommeded to refer the connect to the broker section). Remember to copy and paste these credentials without any spaces in the .env file.
11. **Configure OpenAlgo:** Open the "OpenAlgo" folder in VS Code. Inside the "OpenAlgo" folder, you'll find a ".env" file. This file stores your broker configuration. Replace the placeholder values for API key, API Secret key, and the redirect URL with your actual Broker credentials.
12. **Launch Amibroker:** Download Amibroker from [amibroker.com](https://aistudio.google.com/app/amibroker.com). Install it and get the openalgo amibroker module from openalgo documentation section as shown in the video.
13. **Test your Setup:** You are now ready to test your setup. Open Amibroker and ensure that you have the correct data downloaded and configured. Place a test order using the "Market Order" option. You should be able to see the order ID, timestamp, and other details in your order book and OpenAlgo's trading dashboard.

Remember, if you encounter any issues or have questions, the OpenAlgo community is a great resource. Utilize their Discord server or the documentation on OpenAlgo's website for assistance.

By following these steps, you can create a stable and secure environment on AWS for backtesting, developing, and implementing your own algorithmic trading strategies using OpenAlgo and Amibroker. Happy trading!
