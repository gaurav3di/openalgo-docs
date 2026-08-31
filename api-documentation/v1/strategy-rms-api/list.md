# List Strategies

List the strategies owned by the supplied API key, newest first.

```http
POST /api/v1/strategy/list
Content-Type: application/json
```

```json
{
  "apikey": "<your_app_apikey>",
  "status": "running",
  "q": "NIFTY"
}
```

## Request body

| Field | Required | Rules |
|---|---|---|
| `apikey` | Yes | OpenAlgo API key |
| `status` | No | `stopped`, `running`, `paused`, or `errored` |
| `q` | No | Case-insensitive name substring, at most 100 characters |

`status` and `q` can be omitted or sent as `null` for no filter.

## Response

```json
{
  "status": "success",
  "data": [
    {
      "id": 7,
      "name": "NIFTY Short Straddle",
      "strategy_kind": "batch",
      "strategy_type": "intraday",
      "product": "NRML",
      "pricetype": "MARKET",
      "overall_sl_mtm": -5000.0,
      "overall_target_mtm": 8000.0,
      "live_enabled": false,
      "status": "running",
      "current_run_id": 42,
      "created_at": "2026-08-30T03:50:11.402118+00:00"
    }
  ]
}
```

Each row includes the saved strategy configuration, scheduler and risk settings, live and webhook state, and timestamps. It omits leg configuration; call [Strategy Status](status.md) for that. `product` is the configured intent: the order record shows the venue-specific product actually sent.

No response contains a webhook token. Only its digest is stored.

---

**Next**: [Strategy Status](status.md) | **Back to**: [Strategy RMS API](README.md)
