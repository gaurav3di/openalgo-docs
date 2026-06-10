# Upgrade

Follow the steps for **your installation method**. The procedure differs depending on how you installed OpenAlgo.

{% hint style="danger" %}
**Never run `cp .sample.env .env` on an existing installation.** It erases your broker keys and replaces your `API_KEY_PEPPER` — which permanently invalidates every stored password hash and encrypted broker token. Your `.env` is not tracked by git, so `git pull` always preserves it. When an update introduces new environment variables, copy **only those new lines** from `.sample.env` into your existing `.env` (see "Handling new environment variables" below).
{% endhint %}

***

### Option 1: Ubuntu Server (installed via install.sh)

Your installation lives at `/var/python/openalgo` and runs as the systemd service `openalgo`. Use the bundled update script — it performs the entire upgrade safely in one command:

```bash
cd /var/python/openalgo
sudo bash install/update.sh
```

The script automatically:

1. Stops the `openalgo` service
2. **Backs up all databases** (openalgo, logs, latency, sandbox, historify) to a timestamped `db/backup_<timestamp>/` folder
3. Pulls the latest code (`git pull`)
4. Compares your `.env` against `.sample.env` and **lists any new variables** you should add (it never overwrites your `.env`)
5. Updates Python dependencies
6. Runs all database migrations
7. Restarts the service

Legacy multi-deployment installs (`/var/python/openalgo-flask/<name>/`) are detected automatically and the correct `openalgo-<name>` service is updated.

***

### Option 2: Docker (with or without custom domain)

```bash
# From the folder where you cloned openalgo
cd ~/openalgo

# Pull the latest code (includes the pre-built frontend)
git pull

# Rebuild and restart the container
sudo docker compose down
sudo docker compose build --no-cache
sudo docker compose up -d

# Watch startup logs
sudo docker compose logs -f
```

Your `.env` file and the `db/` volume persist across the rebuild — no reconfiguration needed. Database migrations run automatically on container startup.

***

### Option 3: Desktop / Local — Update Scripts (Recommended)

OpenAlgo runs cross-platform — Windows, macOS and Linux desktops all have a one-command update path using the bundled scripts.

#### Windows

```bat
cd openalgo
install\update.bat
```

#### macOS / Linux

```bash
cd openalgo
bash install/update.sh
```

Both scripts detect a local (non-server) installation automatically and perform the same safe sequence: **back up all databases** (openalgo, logs, latency, sandbox, historify) to a timestamped `db\backup_<timestamp>\` folder → `git pull` → report any new `.env` variables → update dependencies via uv → run all database migrations. Your `.env` is never touched.

After the script finishes, start OpenAlgo as usual:

```bash
uv run app.py
```

You should see the configuration check pass and all databases initialize:

```
Configuration version check passed
INFO in telegram_db: Telegram database initialized successfully
INFO in base: Scheduler started
INFO in traffic_db: Initializing Traffic Logs DB at: sqlite:///db/logs.db
INFO in latency_db: Initializing Latency DB at: sqlite:///db/latency.db
INFO in auth_db: Initializing Auth DB
INFO in symbol: Initializing Master Contract DB
```

***

### Option 4: Manual Upgrade (any platform)

If you prefer to run the steps yourself, only the backup step differs by platform — everything else is identical on Windows, macOS and Linux.

#### 1. Backup the databases

**Windows (PowerShell):**

```powershell
cd openalgo
New-Item -ItemType Directory -Force db\backup | Out-Null
Copy-Item db\*.db db\backup\
Copy-Item db\historify.duckdb db\backup\ -ErrorAction SilentlyContinue
```

**macOS / Linux:**

```bash
cd openalgo
mkdir -p db/backup
cp db/*.db db/backup/ 2>/dev/null
cp db/historify.duckdb db/backup/ 2>/dev/null
```

#### 2. Pull the latest application code

```bash
git pull
```

#### 3. Update dependencies

```bash
uv sync
```

(Legacy pip setups: `pip install -r requirements.txt` inside your virtual environment.)

#### 4. Run the migration script

Migrations are idempotent — safe to run on every upgrade:

```bash
uv run upgrade/migrate_all.py
```

#### 5. Start OpenAlgo

```bash
uv run app.py
```

***

### Handling new environment variables

Newer releases often introduce new environment variables that OpenAlgo needs to function. **Do not recreate your `.env` from the sample** — merge instead:

1. Open `.sample.env` and check `ENV_CONFIG_VERSION` at the top. If it is newer than the version in your `.env`, new variables were added.
2. On startup, OpenAlgo's configuration check reports exactly which variables are missing.
3. Copy **only the missing lines** from `.sample.env` into your `.env` and set appropriate values, keeping all your existing settings (broker keys, `APP_KEY`, `API_KEY_PEPPER`, domain configuration) untouched.

The update scripts (`install/update.sh` on Linux/macOS, `install\update.bat` on Windows) print this comparison for you automatically.

{% hint style="warning" %}
**Never copy `APP_KEY` or `API_KEY_PEPPER` values from documentation or `.sample.env` into a real installation.** Generate your own:

```bash
python -c "import secrets; print(secrets.token_hex(32))"
```

And never change `API_KEY_PEPPER` on an installation that already has users or broker logins — it is used to hash passwords and encrypt broker tokens, and rotating it makes that data unrecoverable. If you genuinely need to rotate it, use the dedicated migration: `uv run python upgrade/rotate_pepper.py`.
{% endhint %}

***

### Post-Upgrade Checklist

* Configuration version check passes on startup (no missing variable warnings)
* All databases initialize correctly
* Scheduler, WebSocket proxy (port 8765) and ZeroMQ services are running
* Broker login works and the master contract downloads
* API requests and strategies function normally

If anything fails after an upgrade, check `log/errors.jsonl` first — it contains structured error details including full tracebacks.

####
