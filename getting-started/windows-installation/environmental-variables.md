# Environmental Variables

Rename the `.sample.env` file located in `openalgo` folder to `.env`

Update the `.env` with your specific configurations as shown in the provided template.



### Sample Environmental File



```python
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

Here’s a clear and professional documentation update for your `.sample.env`, suitable for inclusion at:

📄 [**https://docs.openalgo.in/getting-started/windows-installation/environmental-variables**](https://docs.openalgo.in/getting-started/windows-installation/environmental-variables)

***

### ✅ OpenAlgo `.env` Configuration Explained

This file controls OpenAlgo's broker credentials, security options, server settings, and key platform behaviors like rate limits, API access, CORS, and CSP.

***

#### 🔐 Broker Configuration

```env
BROKER_API_KEY = 'YOUR_BROKER_API_KEY'
BROKER_API_SECRET = 'YOUR_BROKER_API_SECRET'

# Market Data Configuration (Optional and Required only for XTS API Supported Brokers)

BROKER_API_KEY_MARKET = 'YOUR_BROKER_MARKET_API_KEY'
BROKER_API_SECRET_MARKET = 'YOUR_BROKER_MARKET_API_SECRET'
```

Set your broker credentials here. Required for placing live orders. For XTS-supported brokers, market keys can be optionally configured.

***

#### 🔄 Redirect URLs

```env
REDIRECT_URL=http://127.0.0.1:5000/<broker>/callback
```

Used in OAuth-based brokers (like Fyers). Update if you're using a custom domain or ngrok.

***

#### ✅ Valid Brokers

```env
VALID_BROKERS=fivepaisa,fyers,zerodha,...
```

Comma-separated list of supported broker plugins enabled in your OpenAlgo deployment.

***

#### 🔒 Application Keys

```env
APP_KEY=...
API_KEY_PEPPER=...
```

* `APP_KEY`: Internal application identifier.
*   `API_KEY_PEPPER`: Random 64-character hex string used for API key hashing, password hashing, and broker token encryption.\
    🔐 Generate it with:

    ```bash
    python -c "import secrets; print(secrets.token_hex(32))"
    ```

***

#### 🗃️ Database

```env
DATABASE_URL=sqlite:///db/openalgo.db
```

Use SQLite locally. You can point to PostgreSQL or MySQL if needed.

***

#### 🌐 Server Configuration

```env
FLASK_HOST_IP=127.0.0.1
FLASK_PORT=5000
FLASK_ENV=development
```

Control local Flask server binding and environment behavior.\
Set `FLASK_HOST_IP=0.0.0.0` if you want OpenAlgo accessible from other devices on your LAN.

***

#### 🚦 Rate Limiting

```env
LOGIN_RATE_LIMIT_MIN="5 per minute"
LOGIN_RATE_LIMIT_HOUR="25 per hour"
API_RATE_LIMIT="10 per second"
```

Protects API and login endpoints from brute-force or abuse.

***

#### ⏱️ Strategy Execution Delay

```env
SMART_ORDER_DELAY=0.5
```

Delay in seconds between multi-legged order execution. Useful for options or basket strategies.

***

#### 🕒 Session Expiry

```env
SESSION_EXPIRY_TIME=03:00
```

Automatically expires all user sessions daily at this time (IST). Format: HH:MM

***

### 🌐 CORS Configuration (Cross-Origin Resource Sharing)

```env
CORS_ENABLED=TRUE
CORS_ALLOWED_ORIGINS=http://127.0.0.1:5000
CORS_ALLOWED_METHODS=GET,POST
CORS_ALLOWED_HEADERS=Content-Type,Authorization,X-Requested-With
CORS_ALLOW_CREDENTIALS=FALSE
CORS_MAX_AGE=86400
```

* CORS is **only required for browser-based apps** (like React/Vue UIs) calling the OpenAlgo API.
* By default, only requests from the same origin are allowed (`http://127.0.0.1:5000`).
* To support external frontends (e.g., `http://localhost:3000` or `https://data.openalgo.in`), you must add those domains to `CORS_ALLOWED_ORIGINS`.

> 🧠 **CORS does not affect Postman, curl, TradingView, or server-side requests.** It's enforced only by browsers.

<figure><img src="../../.gitbook/assets/image (108).png" alt=""><figcaption></figcaption></figure>

***

### 🔐 CSP Configuration (Content Security Policy)

```env
CSP_ENABLED=TRUE
CSP_REPORT_ONLY=FALSE
CSP_DEFAULT_SRC='self'
...
CSP_SCRIPT_SRC='self' https://cdn.socket.io
CSP_STYLE_SRC='self' 'unsafe-inline'
CSP_IMG_SRC='self' data:
CSP_CONNECT_SRC='self' wss: ws:
...
```

* Adds a security header to prevent XSS, data injection, or loading of untrusted resources.
* By default, only your own domain (`'self'`) and essential WebSocket/Socket.IO scripts are allowed.
* Set `CSP_REPORT_ONLY=TRUE` to **test** without breaking your app (recommended for dev).
* Add CSP rules cautiously when embedding iframes (e.g., TradingView), custom fonts, or CDNs.

***

#### Optional CSP Reporting

```env
CSP_REPORT_URI=/csp-report
```

If set, violations are reported here. You can use this to monitor violations via tools like Sentry or self-logging.

***

### 🧪 Recommended Defaults for Local Testing

| Variable               | Suggested Value         |
| ---------------------- | ----------------------- |
| `CORS_ENABLED`         | `TRUE`                  |
| `CORS_ALLOWED_ORIGINS` | `http://127.0.0.1:5001` |
| `CSP_ENABLED`          | `TRUE`                  |
| `CSP_REPORT_ONLY`      | `TRUE`                  |

***

