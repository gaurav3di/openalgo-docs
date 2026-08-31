# Strategy Status

Read one owned Strategy RMS configuration, its legs, and its current run when one exists.

## Endpoint URL

```http
Local Host   :  POST http://127.0.0.1:5000/api/v1/strategy/status
Ngrok Domain :  POST https://<your-ngrok-domain>.ngrok-free.app/api/v1/strategy/status
Custom Domain:  POST https://<your-custom-domain>/api/v1/strategy/status
```

## Sample API Request

```json
{
  "apikey": "<your_app_apikey>",
  "strategy_id": 7
}
```

## Sample cURL Request

```bash
curl -X POST http://127.0.0.1:5000/api/v1/strategy/status \
  -H 'Content-Type: application/json' \
  -d '{
  "apikey": "<your_app_apikey>",
  "strategy_id": 7
}'
```

## Sample API Response

```json
{
  "status": "success",
  "data": {
    "id": 7,
    "name": "NIFTY Short Straddle",
    "strategy_kind": "batch",
    "status": "running",
    "current_run_id": 42,
    "legs": [
      {
        "id": 1,
        "segment": "options",
        "position": "S",
        "lots": 1,
        "option_type": "CE",
        "expiry": "weekly"
      }
    ]
  },
  "run": {
    "id": 42,
    "mode": "sandbox",
    "broker": "sandbox",
    "started_at": "2026-08-30T03:50:11.402118+00:00",
    "stopped_at": null,
    "stop_requested_at": null,
    "stop_requested_reason": null,
    "pnl_realized": 0.0,
    "pnl_peak": 4880.0,
    "pnl_trough": -1220.25,
    "resolved_expiries": {"1": "04-SEP-26"}
  }
}
```

## Request Body

| Parameter | Description | Mandatory/Optional | Default Value |
|---|---|---|---|
| `apikey` | Your OpenAlgo API key | Mandatory | - |
| `strategy_id` | Positive Strategy RMS id | Mandatory | - |

## Response Fields

| Field | Type | Description |
|---|---|---|
| `status` | string | `success` or `error` |
| `data` | object | Strategy configuration, including its saved leg definitions |
| `run` | object or null | Current run, or `null` when no run is current |

### Run Object Fields

| Field | Type | Description |
|---|---|---|
| `id` | integer | Run id |
| `mode` | string | `live` or `sandbox`, fixed for the run |
| `broker` | string | Broker captured at start; `sandbox` for a sandbox run |
| `started_at`, `stopped_at` | string or null | ISO 8601 UTC timestamps |
| `stop_reason` | string or null | Terminal reason once finalised |
| `stop_requested_at`, `stop_requested_reason` | string or null | Durable pending-stop state |
| `pnl_realized` | number | Realised P&L after confirmed-flat finalisation |
| `pnl_peak`, `pnl_trough` | number | Highest and lowest run P&L |
| `trigger_source` | string | `manual`, `webhook`, or `scheduler` |
| `resolved_expiries` | object or null | Resolved expiry by string leg id |

## Notes

- A non-null `stop_requested_at` or `stop_requested_reason` means the run remains open and managed. Do not treat a stop request as proof of flatness.
- On an open run, P&L fields are persisted values rather than a current market mark. Use a finalised [Run History](runs.md) record for realised P&L.
- `resolved_expiries` preserves the contract identity that was chosen at run start.
- A missing strategy and a strategy owned by another user both return HTTP 404 with `Strategy not found`.

---

**Back to**: [Strategy RMS API](README.md)
