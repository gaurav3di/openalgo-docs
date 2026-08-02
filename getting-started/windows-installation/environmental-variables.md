# Environmental Variables

OpenAlgo reads configuration from `.env` in the application directory. Create it from the `.sample.env` shipped with the same OpenAlgo release:

```powershell
Copy-Item .sample.env .env
```

Do not copy an old environment block from documentation or another server. The checked-out `.sample.env` is the authoritative list; the current application template contains 108 keys and declares `ENV_CONFIG_VERSION = '1.0.8'`.

## Required Setup

At minimum, review these groups before starting OpenAlgo:

### Broker Credentials

```dotenv
BROKER_API_KEY = 'YOUR_BROKER_API_KEY'
BROKER_API_SECRET = 'YOUR_BROKER_API_SECRET'
BROKER_API_KEY_MARKET = 'YOUR_XTS_MARKET_API_KEY_IF_REQUIRED'
BROKER_API_SECRET_MARKET = 'YOUR_XTS_MARKET_API_SECRET_IF_REQUIRED'
REDIRECT_URL = 'http://127.0.0.1:5000/<broker>/callback'
```

The market-data credentials are used only by broker integrations that require a separate XTS market-data login. Replace `<broker>` in `REDIRECT_URL` with the active plugin's callback path and use the public HTTPS host for remote deployments.

### Security Secrets

```dotenv
APP_KEY = 'GENERATE_A_RANDOM_VALUE'
API_KEY_PEPPER = 'GENERATE_A_DIFFERENT_RANDOM_VALUE'
FERNET_SALT = 'GENERATE_A_RANDOM_SALT'
```

Never reuse values copied from examples, `.sample.env`, another installation, or a public repository. The maintained installers generate per-install secrets. If configuring manually, generate strong independent random values and keep `.env` private.

### Application and Realtime URLs

```dotenv
HOST_SERVER = 'http://127.0.0.1:5000'
FLASK_HOST_IP = '127.0.0.1'
FLASK_PORT = '5000'
WEBSOCKET_HOST = '127.0.0.1'
WEBSOCKET_PORT = '8765'
WEBSOCKET_URL = 'ws://127.0.0.1:8765'
ZMQ_HOST = '127.0.0.1'
ZMQ_PORT = '5555'
```

For a public deployment, terminate TLS at the supported reverse proxy, set `HOST_SERVER` to the public `https://` URL, and expose the raw WebSocket proxy through the documented `/ws` route. Keep ZeroMQ on loopback; it is not an authenticated public protocol.

### Database Paths

```dotenv
DATABASE_URL = 'sqlite:///db/openalgo.db'
LATENCY_DATABASE_URL = 'sqlite:///db/latency.db'
LOGS_DATABASE_URL = 'sqlite:///db/logs.db'
HEALTH_DATABASE_URL = 'sqlite:///db/health.db'
SANDBOX_DATABASE_URL = 'sqlite:///db/sandbox.db'
HISTORIFY_DATABASE_PATH = 'db/historify.duckdb'
```

Historify uses `HISTORIFY_DATABASE_PATH`. The obsolete `HISTORIFY_DATABASE_URL` spelling is not an alias and should be replaced in older `.env` files.

## Broker Allowlist

`VALID_BROKERS` is a comma-separated list of enabled plugin keys. The release template contains all 35 installed plugins:

```dotenv
VALID_BROKERS = 'aliceblue,angel,arrow,compositedge,definedge,deltaexchange,dhan,dhan_sandbox,firstock,fivepaisa,fivepaisaxts,flattrade,fyers,groww,hdfcsky,ibulls,iifl,iiflcapital,indmoney,jainamxts,kotak,motilal,mstock,nubra,paytm,pocketful,rmoney,samco,shoonya,tradejini,tradesmart,upstox,wisdom,zebu,zerodha'
```

Broker presence does not guarantee every optional exchange, GTT, historical-data, or streaming capability. See [Brokers](../../connect-brokers/brokers/README.md).

## Updating an Existing Installation

1. Back up `.env` securely.
2. Compare its `ENV_CONFIG_VERSION` with the new `.sample.env`.
3. Add new keys and review changed defaults; do not overwrite working broker credentials or generated secrets.
4. Replace obsolete names such as `HISTORIFY_DATABASE_URL` with their current counterparts.
5. Restart OpenAlgo and read startup validation errors before enabling automation.

Do not commit `.env`, paste it into an issue, or include it in a screenshot. It contains broker credentials, application secrets, and deployment policy.
