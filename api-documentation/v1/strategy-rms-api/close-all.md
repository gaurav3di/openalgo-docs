# Close All Legs

Record an operator close-all intent, persist a whole-run stop, and submit MARKET exits for every owned position in the current run.

```http
POST /api/v1/strategy/close_all
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

The mechanics and state guarantees are the same as [Stop Run](stop.md). The difference is audit intent: this route writes `close_all_manual`, with the message `Operator requested closure of all held legs`, before broker exits settle. That event proves an operator requested a close; it does not prove that the broker is flat.

Read `stop_pending` and each `exits[].ok` value. A refused exit leaves the run open and managed, and returns 409 rather than falsely closing exposure. Use the later `run_stopped` event or a finalised [Run History](runs.md) row as confirmed-flat evidence.

---

**Related**: [Stop Run](stop.md) | [Risk Event Audit Trail](events.md) | **Back to**: [Strategy RMS API](README.md)
