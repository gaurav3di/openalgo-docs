# Start Run

Start a `batch` strategy and place its configured entry legs. Signal strategies do not use this endpoint: their first accepted webhook signal starts the session run.

```http
POST /api/v1/strategy/start
Content-Type: application/json
```

```json
{
  "apikey": "<your_app_apikey>",
  "strategy_id": 7,
  "mode": "sandbox"
}
```

## Request body

| Field | Required | Rules |
|---|---|---|
| `apikey` | Yes | OpenAlgo API key |
| `strategy_id` | Yes | Positive strategy id |
| `mode` | Yes | Exact, case-sensitive `sandbox` or `live`; no default |

## Response

```json
{
  "status": "success",
  "run_id": 42,
  "mode": "sandbox",
  "legs": [
    {
      "leg_id": 1,
      "ok": true,
      "acknowledged": true,
      "symbol": "NIFTY04SEP2624500CE",
      "broker_order_id": "26083004118201",
      "error": null
    }
  ]
}
```

`ok` says whether the broker accepted that entry. A partial start is still HTTP 200, so check every leg outcome. `acknowledged: false` with `ok: true` is not a broker rejection: the order was accepted but its acknowledgement could not be persisted after retry, and a critical reconciliation event remains.

Live mode is refused with 409 until the operator enables live trading on the browser strategy page. Omitting `mode`, or sending values such as `LIVE` or `paper`, returns 400 and reaches no broker. A second start of a current run returns 409 rather than silently placing another set of entries.

If every entry is rejected, the engine closes the empty run and returns 400. Long legs are placed before short legs to avoid a spread's short margin being evaluated before its protective long.

---

**Next**: [Stop Run](stop.md) | **Back to**: [Strategy RMS API](README.md)
