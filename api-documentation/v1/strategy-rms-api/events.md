# Risk Event Audit Trail

Read the append-only risk and lifecycle events for an owned strategy, newest first.

```http
POST /api/v1/strategy/events
Content-Type: application/json
```

```json
{
  "apikey": "<your_app_apikey>",
  "strategy_id": 7,
  "run_id": 42,
  "severity": "critical",
  "limit": 100
}
```

## Request body

| Field | Required | Rules |
|---|---|---|
| `apikey` | Yes | OpenAlgo API key |
| `strategy_id` | Yes | Positive strategy id |
| `run_id` | No | Positive id; must belong to this strategy to match rows |
| `kind` | No | A documented strategy event kind |
| `severity` | No | `info`, `warn`, or `critical` |
| `limit` | No | Integer 1 through 1000; default 500 |

An out-of-vocabulary event kind or severity, or an out-of-range limit, returns 400 rather than an empty or truncated response.

## Response

```json
{
  "status": "success",
  "data": [
    {
      "id": 2041,
      "run_id": 42,
      "strategy_id": 7,
      "ts": "2026-08-30T06:21:40.104112+00:00",
      "kind": "overall_target_hit",
      "severity": "info",
      "leg_id": null,
      "message": "overall target reached",
      "payload": null
    }
  ]
}
```

`payload` is free-form JSON and its shape differs by `kind`; do not make a client depend on one universal payload structure. Configuration events can have `run_id: null`, so a run filter excludes them.

## Events that need attention

| Event | Severity | Meaning |
|---|---|---|
| `run_stop_requested` | info | Stop intent is durable and new signal entries are gated; it is not proof of flatness. |
| `run_stopped` | info | Confirmed-flat terminal transition. |
| `run_stop_failed` | critical | An unfilled entry, refused exit, or terminal exit failure left the run open and managed for retry. |
| `order_ack_unrecorded` | critical | The broker accepted an order but acknowledgement persistence failed after retry; structured repair data is retained. |
| `leg_expiry_fallback` | warn | A nearer expiry was used because the configured expiry rank was unavailable. |
| `flip_outgoing_exit_rejected` | critical | The outgoing side of a signal flip remains held under its exact owner. |
| `close_all_manual` | info | An operator requested close-all; not evidence of a completed close. |

The event log is evidence of what the engine saw and decided at the time. For P&L, use finalised [Run History](runs.md) and durable order/fill evidence rather than treating a live risk event as a settlement record.

---

**Back to**: [Strategy RMS API](README.md)
