# Version 2.0.0.6 Released

**Date: 27th Apr 2026**

**Security Release: APP\_KEY / API\_KEY\_PEPPER Hardening, Encrypted Credentials at Rest, Docker Permission Fix & Broker Updates**

This is a security-focused release covering **9 commits** since v2.0.0.5. It closes the highest-impact remote attack surface in OpenAlgo's web tier (forge-able session cookies signed with the publicly-known sample `APP_KEY`), encrypts every remaining plaintext credential in the database, fixes the Docker `.env` permission bug behind issue #960, and patches a transitive `postcss` CVE pulled in by the React build.

> **Every deployment must run a one-time security upgrade after pulling.** It takes about five minutes, your authenticator codes still work, and your broker session is preserved. See the **Required Upgrade Steps** at the bottom.

***

**Highlights**

* **Per-instance APP\_KEY auto-rotation** — `.sample.env` no longer ships real-looking hex values for `APP_KEY` / `API_KEY_PEPPER`. Instead it carries placeholder strings; the app detects them on first run and replaces them in `.env` with cryptographically random values via `secrets.token_hex(32)`. This closes the entire class of "forge a session cookie offline using the public sample key" attacks against any deployment that previously copied `.sample.env` to `.env` without manually rotating.
* **Credentials encrypted at rest** — TOTP secrets, the Telegram bot token, the Samco 2FA secret, and Flow workflow API keys are now Fernet-encrypted in the database, matching the existing pattern used for broker tokens and TradingView API keys. The Flask session cookie also no longer carries the user's TOTP secret (it was signed but not encrypted, exposing the seed via cookie reads).
* **One-step security upgrade tool** (`upgrade/rotate_pepper.py`) — a single dedicated command rotates `API_KEY_PEPPER` to a fresh value and re-encrypts every credential in the database. Your authenticator codes, broker session, and TradingView/Chartink API keys all keep working. You set a new login password once after running it. Required for every deployment.
* **Docker `.env` permission fix (closes #960)** — fresh Docker installs no longer crash-loop with `Error: .env file not found.` because the install scripts now use a permission mode that the container's `appuser` can read via the bind mount. The Profile → System health check is Docker-aware and recommends the correct mode for each deployment type.
* **postcss CVE-2026-41305** — pinned to `>=8.5.10` (resolves to 8.5.12) via npm `overrides`. Closes Dependabot alert #153.
* **Two earlier credential-leak vectors** — fixes carried in `2c53573f` (closes session and log leaks identified in the security audit).
* **Broker fixes** — Paytm and Groww decimal-strike option symbols (#908), sandbox `FundManager._lock` self-deadlock when the funds row is missing (#1274).

***

**Security**

**APP\_KEY / API\_KEY\_PEPPER**

* `.sample.env` now uses placeholder strings instead of real-looking hex values. New users running `cp .sample.env .env` followed by `uv run app.py` see a one-time green `[OpenAlgo first-run setup]` message; the app generates fresh secrets and writes them back to `.env` atomically.
* Existing users who installed via `install.sh` (custom keys generated at install time) see no change — the check is a single `frozenset` lookup that returns immediately for any non-placeholder value.
* The same detection covers `.env` files copied from a pre-fix commit (the historical leaked literals are also recognised and rotated).
* `update.sh` and `update.bat` no longer fall through to a `cp .sample.env .env` that leaves placeholder values in place; they now generate fresh keys in that recovery branch.
* `.env` is tightened to mode `0o600` on bare-metal installs and `0o644` on Docker installs (UID 1000 must read it via the bind mount).

**Encrypted-at-rest credentials**

Four columns previously stored in plaintext are now Fernet-encrypted on write and decrypted on read:

* `users.totp_secret`
* `bot_config.token` (Telegram bot token)
* `auth.secret_api_key` (Samco 2FA secret)
* `flow_workflows.api_key` (Flow workflow OpenAlgo API key)

A backward-compatible read path means existing rows continue to work between the code update and the security upgrade tool — no in-between broken state.

The Flask session cookie no longer carries `session["totp_secret"]` (Flask sessions are signed, not encrypted; the cookie was leaking the TOTP seed to anyone who could read the cookie value).

**Postcss XSS (CVE-2026-41305)**

* Bumped via `frontend/package.json` `overrides` to `postcss >= 8.5.10`. Resolves to 8.5.12 in the lockfile.
* `npm install` now reports 0 vulnerabilities.

**Pip CVE-2026-3219**

* Dependabot alert #152 dismissed as `tolerable_risk`. `pip` is transitive in `uv.lock`, only invoked at install time, never at runtime. The advisory has no `first_patched_version` published yet; will revisit when `uv` picks up the upstream fix.

***

**Bug Fixes**

**Docker / Install**

* **`.env` permission bug (#960)** — host `.env` at mode `0o600` was unreadable to the container's `appuser` (UID 1000) when bind-mounted, causing `start.sh` to exit with `Error: .env file not found.` and crash-loop the container. Fixed across `install/install-docker.sh`, `install/install-docker-multi-custom-ssl.sh`, and `install/docker-run.sh` (now use mode `0o644`). The Profile → System health check at `/api/system` is Docker-aware and expects the correct mode per deployment.
* **`update.sh` recovery branch** — when `.env` was missing, the script previously copied `.sample.env` verbatim and exited; the running app would have hit the public sample keys. Now generates fresh keys via `secrets.token_hex(32)` in that path.

**Brokers**

* **Paytm and Groww decimal strikes (#908)** — option symbols like `VEDL25APR24292.5CE` now retain the `.5` instead of being silently stripped during symbol normalisation.
* **Kotak — native MPP via `mp` parameter (#1293)** — replace the local `MKT → LMT` rewrite with the broker's native market-protection-price flag.

**Sandbox**

* **`FundManager._lock` self-deadlock (#1274)** — when the sandbox funds row was missing, the `_lock`-holding code path tried to acquire the same lock again. Fix unwinds the call so the missing-row branch releases the lock first.

**Auth / Logging**

* **Two credential-leak vectors closed in `2c53573f`** — session and log paths that could surface API keys / tokens in error contexts.

***

**Frontend / Tooling**

* `frontend/package.json` overrides extended with `postcss >= 8.5.10`.
* `db/backups/` added to `.gitignore` (created automatically by the security upgrade tool).

***

**Dependencies**

* `postcss`: pinned `>= 8.5.10` (resolved 8.5.12) — closes CVE-2026-41305 / GHSA-qx2v-qp2m-jg93.

***

**Required Upgrade Steps**

This release changes the way credentials are protected in your database. **Every deployment must complete the steps below once after pulling**, regardless of how you originally installed.

The whole sequence takes about five minutes. Your data, broker session, authenticator codes, and TradingView API keys are all preserved. The only thing that changes is your login password — you set a new one at the end.

#### 1. Stop OpenAlgo

| Deployment           | Command                                          |
| -------------------- | ------------------------------------------------ |
| Bare-metal (systemd) | `sudo systemctl stop openalgo-<your-deployment>` |
| Docker               | `cd /opt/openalgo && sudo docker compose down`   |
| Docker desktop       | `./install/docker-run.sh stop`                   |
| Local dev            | Ctrl+C in the terminal running `uv run app.py`   |

#### 2. Pull the latest code and dependencies

```bash
cd /path/to/openalgo
git pull origin main
uv sync
```

If you're on Docker and your `.env` was at mode `0o600`, run this once to unbreak the container before the next start:

```bash
sudo chmod 644 .env
```

#### 3. Apply standard database migrations

This is the same step you've always run on every release.

```bash
cd upgrade
uv run migrate_all.py
cd ..
```



#### 4. Restart OpenAlgo

| Deployment           | Command                                           |
| -------------------- | ------------------------------------------------- |
| Bare-metal (systemd) | `sudo systemctl start openalgo-<your-deployment>` |
| Docker               | `cd /opt/openalgo && sudo docker compose up -d`   |
| Docker desktop       | `./install/docker-run.sh start`                   |
| Local dev            | `uv run app.py`                                   |

#### What's preserved (no action needed)

* **Authenticator app codes** — your TOTP secret was rotated under the hood, not regenerated. The same QR you scanned at setup keeps producing valid codes.
* **Broker session** — your saved broker auth token was re-encrypted with the new key. No need to re-OAuth with your broker.
* **TradingView / Chartink / Excel / Python API keys** — your OpenAlgo API key value is unchanged; only its at-rest encryption was refreshed. External integrations continue to work without configuration changes.
* **All trades, positions, strategies, watchlists, settings, logs.**

#### What changes (one-time)

* You set a new login password (Step 6 above).
* Active browser sessions need to log in again.

#### If something goes wrong

The security upgrade tool creates a timestamped backup before any change. To roll back:

```bash
ls db/backups/                          # find your backup
cp db/backups/openalgo.db.before-rotate-pepper-<timestamp> db/openalgo.db
# Then restore the previous API_KEY_PEPPER value in .env from your own backup,
# or contact support on Discord.
```

***

**Contributors**

* **@marketcalls (Rajandran)** — release management, broker fixes (Paytm/Groww strike preservation, Kotak native MPP), sandbox `FundManager` fix, websocket polish, and the v2.0.0.6 security release including APP\_KEY/PEPPER hardening, encrypted-at-rest credential overhaul, the security upgrade tool, the Docker `.env` permission fix, and the postcss CVE bump.

***

**Upgrade Quick Reference**

```bash
# 1. Stop OpenAlgo (see table above for your deployment type)

# 2. Pull and sync
cd /path/to/openalgo
git pull origin main
uv sync

# 3. Run standard migrations
cd upgrade && uv run migrate_all.py && cd ..

# 4. Restart OpenAlgo (see table above for your deployment type)
uv run app.py

# 5. Open the web UI -> Login
```

***

**Links**

* **Repository**: [https://github.com/marketcalls/openalgo](https://github.com/marketcalls/openalgo)
* **Documentation**: [https://docs.openalgo.in](https://docs.openalgo.in)
* **Discord**: [https://www.openalgo.in/discord](https://www.openalgo.in/discord)
* **YouTube**: [https://www.youtube.com/@openalgo](https://www.youtube.com/@openalgo)
* **Issue tracker**: [https://github.com/marketcalls/openalgo/issues](https://github.com/marketcalls/openalgo/issues)

***
