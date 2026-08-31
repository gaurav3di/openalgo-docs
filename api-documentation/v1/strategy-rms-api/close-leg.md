# Close One Leg

Submit a MARKET exit for one filled owner in the current run. Other legs remain running.

```http
POST /api/v1/strategy/close_leg
Content-Type: application/json
```

```json
{
  "apikey": "<your_app_apikey>",
  "strategy_id": 7,
  "leg_id": 2
}
```

## Request body

| Field | Required | Rules |
|---|---|---|
| `apikey` | Yes | OpenAlgo API key |
| `strategy_id` | Yes | Positive strategy id |
| `leg_id` | Yes | Positive leg id from `data.legs[].id` on [Strategy Status](status.md) |

## Response

```json
{
  "status": "success",
  "run_id": 42,
  "leg_id": 2,
  "run_stopped": false,
  "exits": [
    {
      "leg_id": 2,
      "ok": true,
      "position_ref": "80bb5fc9333f4922a582229f06a0fe45",
      "exit_owner": "live",
      "error": null
    }
  ]
}
```

`leg_id` is a configuration leg id, not an order id or a run id. `run_stopped: true` appears only if this call already observed the last owner fill and terminal finalisation completed; a normal asynchronous broker acknowledgement returns `false`.

An exit refusal is an HTTP 409 and `exits[].ok: false`, not a completed close. It remains retryable. A non-open leg, current-run absence, and an accepted-but-unfilled entry are also safe 409 states; none causes an opposing configured-size order to be sent.

---

**Related**: [Stop Run](stop.md) | [Strategy Status](status.md) | **Back to**: [Strategy RMS API](README.md)
