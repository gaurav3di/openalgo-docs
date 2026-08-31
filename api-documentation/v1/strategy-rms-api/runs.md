# Run History

Read every activation of an owned strategy, newest first.

```http
POST /api/v1/strategy/runs
Content-Type: application/json
```

```json
{
  "apikey": "<your_app_apikey>",
  "strategy_id": 7,
  "limit": 100
}
```

## Request body

| Field | Required | Rules |
|---|---|---|
| `apikey` | Yes | OpenAlgo API key |
| `strategy_id` | Yes | Positive strategy id |
| `limit` | No | Integer from 1 through 500; default 100 |

The limit is rejected outside its range rather than silently clamped.

## Response

```json
{
  "status": "success",
  "data": [
    {
      "id": 42,
      "strategy_id": 7,
      "mode": "sandbox",
      "broker": "sandbox",
      "started_at": "2026-08-30T03:50:11.402118+00:00",
      "stopped_at": "2026-08-30T09:40:02.771905+00:00",
      "stop_reason": "overall_target",
      "stop_requested_at": null,
      "stop_requested_reason": null,
      "pnl_realized": 3140.5,
      "pnl_peak": 4880.0,
      "pnl_trough": -1220.25,
      "trigger_source": "manual",
      "resolved_expiries": {"1": "04-SEP-26"}
    }
  ]
}
```

`stopped_at: null` identifies an open run. A non-null `stop_requested_at` and `stop_requested_reason` identify a durable pending stop: new signal entries are gated and recovery resumes the stop until the owner is confirmed flat.

`pnl_realized` is written at confirmed-flat finalisation, then reconciled from fill evidence where necessary. `pnl_peak` and `pnl_trough` are authoritative only after a run stops. A run ending with `stop_reason: "overall_target"` can have a lower realised P&L because risk is evaluated from rolling latest-known marks and MARKET exits can fill later at different prices.

Use `id` as the optional `run_id` filter for [Order History](orders.md) and [Risk Event Audit Trail](events.md).

---

**Back to**: [Strategy RMS API](README.md)
