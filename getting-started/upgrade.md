# Upgrade

Follow these steps to upgrade your OpenAlgo application:

## Option 1 : Regular Upgrade Method

### Backup the Database

Navigate to your database folder and make a copy of the current database file for safekeeping.

```
# Example: Copy the existing DB file to a backup location
cd openalgo
cp db/openalgo.db db/openalgo_backup.db
```

### Update the Application Code

Open your terminal and run the following command to pull the latest changes:

```bash
git pull
```

### Update the .env files

update your .env files from .sample.env (as some of the newer updates often has new enviromentatal variables. which is vital for the openalgo to function

```
cp .sample.env .env
```

and config the apikeys and apisecret as per the openalgo broker documentation

**Install Dependencies**

Install or update the required Python packages by running:

```bash
pip install -r requirements.txt
```

### Run the Migration Script

Execute the migration script to add the `feed_token` column to your database:

<pre class="language-bash"><code class="lang-bash"># Navigate to your OpenAlgo upgrade directory
<strong>cd upgrade
</strong>
# Run the migration script
python migrate_telegram_bot.py
python migrate_smtp_simple.py

# Get back to openalgo directory
cd..

# Run the OpenAlgo App
python app.py
</code></pre>

## Option 2 : UV Method (Modern)

Here is a **separate and clean upgrade guide for the UV method** only:

***

#### 1️⃣ Backup the Database

Before making any changes, **backup your database**:

```bash
# Example: Copy the existing DB file to a backup location
cd openalgo
cp db/openalgo.db db/openalgo_backup.db
```

***

#### 2️⃣ Pull the Latest Application Code

Update to the latest version of OpenAlgo:

```bash
git pull
```

***

#### 3️⃣ Update `.env` File

Copy the latest sample `.env` and edit with your API keys and secrets:

```bash
cp .sample.env .env
```

Then configure the required environment variables inside `.env` as per your broker and setup.

***

#### 4️⃣ Install or Update Dependencies

Use `uv` to install/update packages:

```bash
uv pip install -r requirements.txt
```

***

#### 5️⃣ Run the Migration Script

Update the database schema:

```bash
cd upgrade
uv run migrate_telegram_bot.py
uv run migrate_smtp_simple.py
cd ..
```

***

#### 6️⃣ Run OpenAlgo Using UV

From the root OpenAlgo folder:

```bash
uv run app.py
```

You should see logs like:

```
✅ Configuration version check passed (1.0.3)
[2025-07-31 01:45:12,486] INFO in base: Scheduler started
[2025-07-31 01:45:12,587] INFO in base: Scheduler started
[2025-07-31 01:45:12,589] INFO in market_data_service: MarketDataService initialized
[2025-07-31 01:45:12,927] INFO in traffic_db: Initializing Traffic Logs DB at: sqlite:///db/logs.db
[2025-07-31 01:45:12,939] INFO in latency_db: Initializing Latency DB at: sqlite:///db/latency.db
[2025-07-31 01:45:13,024] INFO in auth_db: Initializing Auth DB
[2025-07-31 01:45:13,025] INFO in user_db: Initializing User DB
[2025-07-31 01:45:13,026] INFO in symbol: Initializing Master Contract DB
[2025-07-31 01:45:13,026] INFO in apilog_db: Initializing API Log DB
[2025-07-31 01:45:13,026] INFO in analyzer_db: Initializing Analyzer Table
...
```

***

#### Post-Upgrade Checklist

* All databases initialize correctly
* Scheduler, Websockets, ZeroMQ services are running
* Broker modules are visible
* API and strategy functions work normally



### Verify the Migration

After running the script, verify that the column was added successfully by:

1. Checking the script output for success messages
2. Logging into your OpenAlgo application and testing broker authentication





### Dependencies Check (Optional)

Ensure all installed dependencies are compatible with the new version:

• Use a virtual environment to avoid conflicts:

```bash
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install -r requirements.txt
```



### Configure the Environment

Create a .env file from the provided .sample.env file. Update the following configurations in the .env file:

• Generate random values for APP\_KEY and API\_KEY\_PEPPER using:\
\
• Update the following parameters:

• BROKER\_API\_KEY

• BROKER\_API\_SECRET

• REDIRECT\_URL

• Verify and adjust other configurations as needed.



#### Sample .env template

```bash
# OpenAlgo Environment Configuration File
# Version: 1.0.3
# Last Updated: 2025-07-31
# 
# IMPORTANT: When updating OpenAlgo, compare this version with your .env file
# If versions don't match, copy new variables from this file to your .env
ENV_CONFIG_VERSION = '1.0.3'

# Broker Configuration
BROKER_API_KEY = 'YOUR_BROKER_API_KEY'
BROKER_API_SECRET = 'YOUR_BROKER_API_SECRET'

# Market Data Configuration (Optional and Required only for XTS API Supported Brokers)

BROKER_API_KEY_MARKET = 'YOUR_BROKER_MARKET_API_KEY'
BROKER_API_SECRET_MARKET = 'YOUR_BROKER_MARKET_API_SECRET'

REDIRECT_URL = 'http://127.0.0.1:5000/<broker>/callback'  # Change if different

# Valid Brokers Configuration

VALID_BROKERS = 'fivepaisa,fivepaisaxts,aliceblue,angel,compositedge,dhan,dhan_sandbox,firstock,flattrade,fyers,groww,ibulls,iifl,indmoney,kotak,paytm,pocketful,shoonya,tradejini,upstox,wisdom,zebu,zerodha'


# Security Configuration
# IMPORTANT: Generate new random values for both keys during setup!

# OpenAlgo Application Key
APP_KEY = '3daa0403ce2501ee7432b75bf100048e3cf510d63d2754f952e93d88bf07ea84'

# Security Pepper - Used for hashing/encryption of sensitive data
# This is used for:
# 1. API key hashing
# 2. User password hashing
# 3. Broker auth token encryption
# Generate a new random string during setup using: python -c "import secrets; print(secrets.token_hex(32))"
API_KEY_PEPPER = 'a25d94718479b170c16278e321ea6c989358bf499a658fd20c90033cef8ce772'

# OpenAlgo Database Configuration
DATABASE_URL = 'sqlite:///db/openalgo.db'

# Additional Database Configuration
LATENCY_DATABASE_URL = 'sqlite:///db/latency.db'  # Database for latency monitoring
LOGS_DATABASE_URL = 'sqlite:///db/logs.db'        # Database for traffic logs 

# OpenAlgo Ngrok Configuration
NGROK_ALLOW = 'FALSE' 

# OpenAlgo Hosted Server (Custom Domain Name) or Ngrok Domain Configuration
# Change to your custom domain or Ngrok domain
HOST_SERVER = 'http://127.0.0.1:5000'  

# OpenAlgo Flask App Host and Port Configuration
# For 0.0.0.0 (accessible from other devices on the network)
# Flask Environment - development or production
FLASK_HOST_IP='127.0.0.1'  
FLASK_PORT='5000' 
FLASK_DEBUG='False' 
FLASK_ENV='development'

# WebSocket Configuration
# Use explicit IPv4 address for macOS compatibility
WEBSOCKET_HOST='127.0.0.1'
WEBSOCKET_PORT='8765'
WEBSOCKET_URL='ws://127.0.0.1:8765'

# ZeroMQ Configuration
# Use explicit IPv4 address for macOS compatibility
ZMQ_HOST='127.0.0.1'
ZMQ_PORT='5555'

# Logging configuration
LOG_TO_FILE='False'           # If True, logs are also written to log files in LOG_DIR
LOG_LEVEL='INFO'              # DEBUG, INFO, WARNING, ERROR, CRITICAL
LOG_DIR='log'                 # Directory for log files (relative to project root)
LOG_FORMAT='[%(asctime)s] %(levelname)s in %(module)s: %(message)s'
LOG_RETENTION='14'            # Number of days to retain log files
LOG_COLORS='True'             # Enable/disable colored console output (True/False)
FORCE_COLOR='1'               # Force enable colored output even in non-TTY environments


# OpenAlgo Rate Limit Settings
LOGIN_RATE_LIMIT_MIN = "5 per minute" 
LOGIN_RATE_LIMIT_HOUR = "25 per hour"
RESET_RATE_LIMIT = "15 per hour"
API_RATE_LIMIT="50 per second"
ORDER_RATE_LIMIT="10 per second"
SMART_ORDER_RATE_LIMIT="2 per second"
WEBHOOK_RATE_LIMIT="100 per minute"
STRATEGY_RATE_LIMIT="200 per minute"

# OpenAlgo API Configuration

# Required to give 0.5 second to 1 second delay between multi-legged option strategies
# Single legged orders are not affected by this setting.
SMART_ORDER_DELAY = '0.5'

# Session Expiry Time (24-hour format, IST)
# All user sessions will automatically expire at this time daily
SESSION_EXPIRY_TIME = '03:00'

# OpenAlgo CORS (Cross-Origin Resource Sharing) Configuration
# Set to TRUE to enable CORS support, FALSE to disable
CORS_ENABLED = 'TRUE'

# Comma-separated list of allowed origins (domains)
# Example: http://localhost:3000,https://example.com
# Use '*' to allow all origins (not recommended for production)
CORS_ALLOWED_ORIGINS = 'http://127.0.0.1:5000'

# Comma-separated list of allowed HTTP methods
# Default: GET,POST
CORS_ALLOWED_METHODS = 'GET,POST,DELETE,PUT,PATCH'

# Comma-separated list of allowed headers
# Default Flask-CORS values will be used if not specified
CORS_ALLOWED_HEADERS = 'Content-Type,Authorization,X-Requested-With'

# Comma-separated list of headers exposed to the browser
CORS_EXPOSED_HEADERS = ''

# Whether to allow credentials (cookies, authorization headers)
# Set to TRUE only if you need to support credentials
CORS_ALLOW_CREDENTIALS = 'FALSE'

# Max age (in seconds) for browser to cache preflight requests
# Default: 86400 (24 hours)
CORS_MAX_AGE = '86400'

# OpenAlgo Content Security Policy (CSP) Configuration
# Set to TRUE to enable CSP, FALSE to disable
CSP_ENABLED = 'TRUE'

# Set to TRUE to use Content-Security-Policy-Report-Only mode (testing without blocking)
# This will report violations but not block content
CSP_REPORT_ONLY = 'FALSE'

# Default source directive - restricts all resource types by default
CSP_DEFAULT_SRC = "'self'"

# Script source directive - controls where scripts can be loaded from
# Includes Socket.IO CDN which is required by the application
# 'unsafe-inline' is needed for Socket.IO to function properly
# Cloudflare Insights is used for analytics
CSP_SCRIPT_SRC = "'self' 'unsafe-inline' https://cdn.socket.io https://static.cloudflareinsights.com"

# Style source directive - controls where styles can be loaded from
# 'unsafe-inline' is needed for some inline styles in the application
CSP_STYLE_SRC = "'self' 'unsafe-inline'"

# Image source directive - controls where images can be loaded from
# 'data:' allows base64 encoded images
CSP_IMG_SRC = "'self' data:"

# Connect source directive - controls what network connections are allowed
# Includes WebSocket connections needed for real-time updates
CSP_CONNECT_SRC = "'self' wss: ws:"

# Font source directive - controls where fonts can be loaded from
CSP_FONT_SRC = "'self'"

# Object source directive - controls where plugins can be loaded from
# 'none' disables all object, embed, and applet elements
CSP_OBJECT_SRC = "'none'"

# Media source directive - controls where audio and video can be loaded from
# Allows audio alerts from your domain and potentially CDN sources in the future
CSP_MEDIA_SRC = "'self' data: https://*.amazonaws.com https://*.cloudfront.net"

# Frame source directive - controls where iframes can be loaded from
# If you integrate with TradingView or other platforms, you may need to add their domains
CSP_FRAME_SRC = "'self'"

# Form action directive - restricts where forms can be submitted to
CSP_FORM_ACTION = "'self'"

# Frame ancestors directive - controls which sites can embed your site in frames
# This helps prevent clickjacking attacks
CSP_FRAME_ANCESTORS = "'self'"

# Base URI directive - restricts what base URIs can be used
CSP_BASE_URI = "'self'"

# Set to TRUE to upgrade insecure (HTTP) requests to HTTPS
# Recommended for production environments
CSP_UPGRADE_INSECURE_REQUESTS = 'FALSE'

# URI to report CSP violations to (optional)
# Example: /csp-report
CSP_REPORT_URI = ''

# CSRF (Cross-Site Request Forgery) Protection Configuration
# Set to TRUE to enable CSRF protection, FALSE to disable
CSRF_ENABLED = 'TRUE'

# CSRF Token Time Limit (in seconds)
# Leave empty for no time limit (tokens valid for entire session)
# Example: 3600 = 1 hour, 86400 = 24 hours
CSRF_TIME_LIMIT = ''

# Cookie Names Configuration for Instance Isolation
# Customize these when running multiple OpenAlgo instances to prevent cookie conflicts
# Each instance should have unique cookie names
# Examples: 'instance1_session', 'user1_session', 'app_session', etc.
SESSION_COOKIE_NAME = 'session'
CSRF_COOKIE_NAME = 'csrf_token'


```
