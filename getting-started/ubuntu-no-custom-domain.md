# Ubuntu (No Custom Domain)

OpenAlgo can be installed on an Ubuntu desktop or laptop by preparing the system, downloading the source code, configuring broker credentials, and launching the application locally.

For traders using an Indian broker, static IP configuration is now mandatory for retail algo trading under SEBI regulations. This must be completed before using transactional broker APIs for live order placement.

{% embed url="https://www.youtube.com/watch?v=gqlAefDwAtw" %}

### System Requirements

Make sure the following are available on your Ubuntu system before you begin.

Python 3

Git

UV package manager

Visual Studio Code or any code editor

Internet connection

Broker API credentials

For Indian brokers, a static IPv4 address

### Step 1 Create a Project Folder

Create a new folder on your Ubuntu desktop or laptop for the OpenAlgo project. The folder name can be anything you prefer.

Open this folder in your code editor.

### Step 2 Check Required Software

Open the terminal inside the project folder and confirm that Python 3, Git, and UV are installed on your system.

These are the core requirements needed to run OpenAlgo.

### Step 3 Download the OpenAlgo Source Code

Go to the OpenAlgo portal and copy the GitHub repository URL.

Download the OpenAlgo source code into your project folder.

This gives you the full local project needed for installation and configuration.

### Step 4 Create the Environment File

Inside the downloaded project, locate the sample environment file.

Create a copy of this sample file and rename it as the main environment file.

This file is used to store your broker related settings and credentials.

### Step 5 Configure Broker Credentials

Open the environment file and update the required values.

You will typically need to enter the following details.

Broker name

Redirect URL

Broker API key

Broker API secret

The exact values and process depend on the broker you use.

### Step 6 Generate API Credentials from Your Broker

Log in to your broker’s developer portal and create an app for API access.

Depending on the broker, you may need to provide an app name, redirect URL, and other broker specific information.

After the app is created, copy the API key and API secret and paste them into the environment file.

### Step 7 Configure Static IP for Indian Brokers

If you are using an Indian broker, static IP setup is mandatory for retail algo trading under SEBI regulations.

Your broker must whitelist the fixed IP address from which your orders originate. This applies to transactional APIs used for placing, modifying, cancelling, basket, or split orders.

Important points to note

Orders must originate from a whitelisted static IP

This requirement applies from 1 April 2026 for retail algo traders using broker APIs

Most brokers provide primary and secondary IP slots

IP changes are generally allowed only once per week

Home internet connections often use dynamic IPs and are not reliable for this purpose

A static IPv4 address is required because brokers do not generally support IPv6 whitelisting

For desktop users, one option is to request a static IP from the internet service provider. Another option is to use a VPS, which usually includes a static IP by default.

More details are available on the OpenAlgo static IP page.

### Step 8 Save Changes Properly

After updating the environment file, save it carefully.

Whenever you change the API key, API secret, or related broker settings, save the file and restart the application so the changes take effect.

### Step 9 Start OpenAlgo

Launch OpenAlgo from the project directory.

On the first run, the application will create its environment, install required libraries, and start the local service.

The first launch may take longer than later launches.

OpenAlgo runs locally on port 5000, and you can access it through your browser after startup.

### Step 10 Complete First Time Account Setup

When OpenAlgo opens for the first time, you will be taken to the setup screen.

Create your account using the required details.

Username

Email address

Password

Password confirmation

Use a strong password that meets the platform requirements.

### Step 11 Log In to OpenAlgo

After the initial account setup, log in using your OpenAlgo credentials.

This is your application login and is separate from your broker authentication.

### Step 12 Connect Your Broker Account

After logging in, connect your broker account from within OpenAlgo.

Broker login is usually a manual step. In the workflow described, the broker session remains active until the next day at 3:00 AM, after which you must log in again.

### Step 13 Wait for Platform Initialization

After connecting the broker, OpenAlgo opens the main dashboard.

At this stage, the platform may begin downloading the master contract. Wait until this process finishes completely.

You should only proceed when the dashboard shows that the system is ready.

Do not start automation until the platform indicates readiness.

### Common Mistakes to Avoid

Do not forget to save the environment file after editing credentials

Do not forget to restart the application after changing broker settings

Do not use a dynamic home IP for Indian broker transactional APIs

Do not begin automation before the dashboard is fully ready

Do not assume the same credential setup process applies to every broker

### Conclusion

Installing OpenAlgo on Ubuntu Desktop involves preparing the system, downloading the project, configuring the environment file, adding broker credentials, setting up static IP when required, launching the application, and completing the first time login flow.

For Indian brokers, static IP compliance is now an essential part of setup and must be completed before live API based trading.
