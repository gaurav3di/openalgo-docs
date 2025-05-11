# Environmental Variables

Rename the `.sample.env` file located in `openalgo` folder to `.env`

Update the `.env` with your specific configurations as shown in the provided template.



### Sample Environmental File



```python
# Broker Configuration
BROKER_API_KEY = 'YOUR_BROKER_API_KEY'
BROKER_API_SECRET = 'YOUR_BROKER_API_SECRET'

# Market Data Configuration (Optional and Required only for XTS API Supported Brokers)

BROKER_API_KEY_MARKET = 'YOUR_BROKER_MARKET_API_KEY'
BROKER_API_SECRET_MARKET = 'YOUR_BROKER_MARKET_API_SECRET'

REDIRECT_URL = 'http://127.0.0.1:5000/<broker>/callback'  # Change if different

# Valid Brokers Configuration
VALID_BROKERS = 'fivepaisa,fivepaisaxts,aliceblue,angel,compositedge,dhan,firstock,flattrade,fyers,groww,iifl,kotak,jainam,jainampro,paytm,pocketful,shoonya,upstox,wisdom,zebu,zerodha'

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

# OpenAlgo Rate Limit Settings
LOGIN_RATE_LIMIT_MIN = "5 per minute" 
LOGIN_RATE_LIMIT_HOUR = "25 per hour"
API_RATE_LIMIT="10 per second"

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
CORS_ALLOWED_METHODS = 'GET,POST'

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

