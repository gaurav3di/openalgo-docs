# Stop Run

Persist a stop request and submit MARKET exits for every exact owner in the strategy's current run. The run finalises only after fills prove it is flat.

```http
POST /api/v1/strategy/stop
Content-Type: application/json
```

```json
{
  "apikey": "<your_app_apikey>",
  "strategy_id": 7
}
```

## Response

```json
{
  "status": "success",
  "run_id": 42,
  "stop_pending": true,
  "exits": [
    {
      "leg_id": 1,
      "ok": true,
      "position_ref": "969bc536b1c14d15992f730c2c136d7a",
      "exit_owner": "live",
      "error": null
    }
  ]
}
```

## Rules that callers must observe

- The request names a strategy, not a run. The engine always resolves its current run.
- `stop_pending: true` means exit fills, retries, or reconciliation are still outstanding. The run remains open, subscribed, and managed.
- `ok: false` means an exit was refused. The position remains held and retryable; a success envelope or non-empty `exits` array is not proof of closure.
- An entry accepted but not yet filled is not exited at the configured quantity. The stop reports its refusal and remains managed so a future fill cannot become a naked position.
- An already-working exit is not dispatched twice.

HTTP 200 means the durable stop request was accepted. HTTP 409 means no run is current, or the engine could not carry out a safe stop; inspect the error and then [Strategy Status](status.md) and [Risk Event Audit Trail](events.md). A terminal `run_stopped` event is the confirmed-flat proof.

---

**Related**: [Close All Legs](close-all.md) | [Run History](runs.md) | **Back to**: [Strategy RMS API](README.md)
