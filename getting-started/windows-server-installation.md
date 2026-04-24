# Windows Server Installation

Windows Server Installation (Updated)

Why Choose a Server over a Desktop?

* Reliability and Stability: Servers provide a stable and dedicated environment for your trading software. Unlike a desktop, servers are designed for constant operation and are less susceptible to disruptions like power outages, system crashes, or internet issues.
* Scalability: If your trading algorithms become more complex or require access to vast amounts of data, you can easily scale up your server resources to accommodate those needs.
* Security: Servers offer enhanced security measures to protect your trading strategies and data. You can control access, enforce strict firewall rules, and implement encryption protocols for better data protection.
* Accessibility: You can access your server and trading platform remotely, regardless of your location. This allows for greater flexibility and control over your operations.

This updated guide focuses on a minimal, high-performance installation of OpenAlgo on a Windows Server (Vultr/AWS/Azure), specifically targeting regions like Mumbai for optimal trading latency. It introduces the uv package manager for significantly faster dependency management compared to standard pip.

{% embed url="https://www.youtube.com/watch?v=ZkXZ5yoQkUQ" %}

Step-by-Step Installation Process:

1. Deploy a Cloud Instance: Deploy a Windows Server 2022 instance via your preferred cloud provider. For Indian markets, choose the Mumbai data center. A minimum of 8GB RAM is recommended.
2. Access via RDP: Log into your instance using the Remote Desktop Connection tool with the Administrator credentials provided by your cloud console.
3. Install Google Chrome: Open the default browser (Edge) to download and install Google Chrome for a better browsing experience on the server.
4. Install Visual C++ Redistributable: Download and install the latest VC++ Redistributable (both x86 and x64). This is essential for libraries like Numba, which OpenAlgo uses for high-performance calculations.
5. Install Python: Download the latest stable version of Python from python.org. Important: During installation, ensure you check the box "Add python.exe to PATH".
6. Install Git: Download and install Git for Windows from git-scm.com. This is required to clone and update the OpenAlgo source code.
7. Install uv Package Manager: Open Windows PowerShell as Administrator and run the following command to install uv: pip install uv
8. Download OpenAlgo: Navigate to your C: drive in PowerShell (cd C:) and clone the repository: git clone https://github.com/marketcalls/openalgo
9. Configure .env File:
   * Enter the folder: cd openalgo
   * Create a copy of the sample config: copy .sample.env .env
   * Open the .env file in Notepad and enter your Broker API Key and API Secret. Save and close.
10. Set Server Timezone: For Indian markets, adjust your server time settings to (UTC +05:30) Chennai, Kolkata, Mumbai, New Delhi. Ensure you click "Sync now" to align with exchange timings.
11. Update Static IP at Broker: Copy your server's Public IP address. Log in to your broker's API portal and whitelist this IP in your App settings. This is a critical security step required by most brokers.
12. Initial Launch: Run OpenAlgo for the first time by typing: uv run app.py Note: The first launch will take a few minutes as uv builds the virtual environment and installs 175+ dependencies.

Create a One-Click Launch Script

To simplify the startup process, you can create a PowerShell script on your desktop:

1. Open Notepad and paste the following code:

## Start OpenAlgo using uv on Windows Server 2022

$OpenAlgoPath = "C:\openalgo"

Write-Host "Starting OpenAlgo..." Write-Host "Folder: $OpenAlgoPath"

## Go to OpenAlgo folder

Set-Location $OpenAlgoPath

## Check uv is available

if (-not (Get-Command uv -ErrorAction SilentlyContinue)) { Write-Host "ERROR: uv is not installed or not available in PATH." -ForegroundColor Red exit 1 }

## Run OpenAlgo

uv run app.py

2. Save the file to your Desktop as StartOpenAlgo.ps1.
3. To start OpenAlgo, Right-click the file and select "Run with PowerShell".

