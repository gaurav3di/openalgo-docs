# Strategy Status

Return one owned strategy, its complete leg configuration, and its current run when one exists.

```http
POST /api/v1/strategy/status
Content-Type: application/json
```

```json
{
  "apikey": "<your_app_apikey>",
  "strategy_id": 7
}
```

## Request body

| Field | Required | Rules |
|---|---|---|
| `apikey` | Yes | OpenAlgo API key |
| `strategy_id` | Yes | Positive strategy id |

## Response

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
      {"id": 1, "segment": "options", "position": "S", "lots": 1, "option_type": "CE", "expiry": "weekly"}
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

`data` contains the fields returned by [List Strategies](list.md) plus `legs`. `run` is `null` when there is no `current_run_id`. `resolved_expiries` pins the contracts resolved at start, so a later chain update does not change an existing run's identity.

A populated `stop_requested_at` or `stop_requested_reason` means a durable stop remains pending. The run is still current and managed; wait for `stopped_at` or for `run` to become `null` before treating it as flat.

The P&L columns on an open run are persisted values, not the live mark. Use a finalised row from [Run History](runs.md) for realised P&L. A stopped run has zero unrealised P&L.

---

**Next**: [Start Run](start.md) | **Back to**: [Strategy RMS API](README.md)
