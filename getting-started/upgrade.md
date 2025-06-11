# Upgrade

Follow these steps to upgrade your OpenAlgo application:

### Backup the Database

Navigate to your database folder and make a copy of the current database file for safekeeping.



### Update the Application Code

Open your terminal and run the following command to pull the latest changes:

```bash
cd openalgo
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
python add_feed_token.py
python add_user_id.py

# Get back to openalgo directory
cd..

# Run the OpenAlgo App
python app.py
</code></pre>

The script will:

1. Check if the `feed_token` column already exists
2. Add the column if it doesn't exist
3. Log the results of the operation

### Verify the Migration

After running the script, verify that the column was added successfully by:

1. Checking the script output for success messages
2. Logging into your OpenAlgo application and testing Angel broker authentication

### Technical Notes

#### What is the Feed Token?

The feed token (data token) is provided by Angel Broking's API during authentication and is used for real-time market data operations. It is separate from the authentication token and has its own validity period.

#### Implementation Details

The upgrade adds:

1. A new `feed_token` column to the `auth` table in the database
2. Storage and retrieval mechanisms for the feed token
3. Integration of the feed token with market data operations

#### Modified Files

This upgrade modifies the following key files:

1. `openalgo/broker/angel/api/auth_api.py` - To extract and return the feed token from the Angel authentication response
2. `openalgo/database/auth_db.py` - To add storage and retrieval functions for the feed token
3. `openalgo/utils/auth_utils.py` - To handle the feed token in session management

### Troubleshooting

#### "Unable to open database file" Error

If you encounter this error, check:

1. The database path in your `.env` file is correct
2. The directory exists and has proper permissions
3. You are running the script from the correct location

#### "feed\_token column already exists" Message

This is not an error. It means the migration has already been applied to your database.

#### Authentication Issues After Upgrade

If you experience authentication issues:

1. Verify that all files have been properly updated
2. Log out and log back in to generate a new feed token
3. Check the application logs for specific error messages

### Support

If you need assistance with the feed token upgrade, please:

1. Check the OpenAlgo documentation
2. Refer to the Angel broking API documentation for feed token details
3. File an issue on the OpenAlgo GitHub repository



### Dependecies Check (Optional)

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
# Broker Configuration
BROKER_API_KEY = 'YOUR_BROKER_API_KEY'
BROKER_API_SECRET = 'YOUR_BROKER_API_SECRET'

REDIRECT_URL = 'http://127.0.0.1:5000/<broker>/callback'  # Change if different

# Valid Brokers Configuration
VALID_BROKERS = 'fivepaisa,aliceblue,angel,dhan,fyers,icici,kotak,shoonya,upstox,zebu,zerodha'

# Security Configuration
# IMPORTANT: Generate new random values for both keys during setup!

# OpenAlgo Application Key
APP_KEY = 'GENERATE_A_RANDOM_APP_KEY'

# Security Pepper - Used for hashing/encryption of sensitive data
# This is used for:
# 1. API key hashing
# 2. User password hashing
# 3. Broker auth token encryption
API_KEY_PEPPER = 'GENERATE_A_RANDOM_PEPPER_KEY'

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

# OpenAlgo Flask App Version Management
FLASK_APP_VERSION='1.0.0.14'

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
```
