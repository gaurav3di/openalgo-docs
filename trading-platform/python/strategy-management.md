# Strategy RMS Alerts From Python

Use a normal HTTP client to send alerts to a Strategy RMS public webhook. Create the strategy and copy its webhook token from the `/strategy` page first. The URL token is a credential: store it in a secret manager or environment variable, never in source control or logs.

This is the current Strategy RMS contract. The retired `Strategy(...)` helper, `BUY`/`SELL` modes, `position_size`, and per-symbol mapping protocol do not apply to the rewritten engine.

## Batch strategy example

A batch strategy accepts `start` and `stop`. `mode` is required for `start`; use `sandbox` until the strategy has been explicitly enabled for live trading in the browser.

```python
import os

import requests

base_url = os.environ["OPENALGO_URL"].rstrip("/")
token = os.environ["OPENALGO_STRATEGY_WEBHOOK_TOKEN"]
webhook_url = f"{base_url}/strategy/webhook/{token}"

response = requests.post(
    webhook_url,
    json={"action": "start", "mode": "sandbox"},
    timeout=10,
)
response.raise_for_status()
print(response.json())
```

To request a stop:

```python
response = requests.post(webhook_url, json={"action": "stop"}, timeout=10)
response.raise_for_status()
result = response.json()

if result.get("stop_pending"):
    print("Exit is accepted but not yet confirmed flat; monitor strategy status/events.")
```

A successful stop response does not prove that exits filled. `stop_pending: true` means the run remains managed while the broker fills, retries, or reconciles the exits.

## Signal strategy example

Signal strategies accept `long_entry`, `long_exit`, `short_entry`, and `short_exit`. Address the leg by its configured `leg_id`, or by symbol and exchange.

```python
response = requests.post(
    webhook_url,
    json={"action": "long_entry", "leg_id": 1},
    timeout=10,
)
response.raise_for_status()
print(response.json())
```

The first accepted signal in the platform session opens the run. Repeating an entry that is already held is an accepted no-op, which makes sender retries safe. An action that does not match the strategy kind, its configured direction, or a configured leg is rejected for correction.

## Security And Operations

- The webhook accepts no `apikey`; the URL token is the credential.
- Do not include the token in alert JSON, exception text, request logs, or screenshots.
- Configure a webhook IP allowlist when your sender publishes stable address ranges.
- HTTP 429 is a rate-limit response. Retry only with bounded backoff and avoid sending duplicate business alerts.
- If a token may have leaked, rotate it in the strategy page. The old token stops working immediately.

See [Public Strategy Webhook](../../api-documentation/v1/strategy-rms-api/webhook.md) for every result code and [Strategy RMS RESTX API](../../api-documentation/v1/strategy-rms-api/README.md) for authenticated lifecycle and audit calls.
