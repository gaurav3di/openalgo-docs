# Multi Option Greeks

Calculate Black-76 Greeks and implied volatility for 1 to 50 option symbols. The service batches option quote retrieval, reuses underlying quotes, and uses a per-expiry synthetic forward when available, with spot as fallback.

## Endpoint

```http
POST /api/v1/multioptiongreeks
```

```bash
curl -X POST 'http://127.0.0.1:5000/api/v1/multioptiongreeks' \
  -H 'Content-Type: application/json' \
  -d '{
    "apikey": "<your_app_apikey>",
    "symbols": [
      {"symbol": "NIFTY30JUL2625000CE", "exchange": "NFO"},
      {"symbol": "NIFTY30JUL2625000PE", "exchange": "NFO"}
    ],
    "interest_rate": 7.0,
    "expiry_time": "15:30"
  }'
```

## Request

| Field | Required | Description |
|---|---:|---|
| `apikey` | Yes | OpenAlgo API key |
| `symbols` | Yes | Array containing 1 to 50 option requests |
| `interest_rate` | No | Common annualized rate from 0 to 100 percent |
| `expiry_time` | No | Common expiry time in `HH:MM` form |

These four fields are the complete top-level `MultiOptionGreeksSchema`. Any other top-level field returns HTTP 400. There is no per-item `interest_rate` and no per-item `expiry_time`: both are set once for the whole batch.

Each `symbols` item requires `symbol` and `exchange`. Valid exchanges are the derivatives list: `NFO`, `BFO`, `MCX`, `CDS`, `NCO`, `BCD`, `NCDEX`, and `CRYPTO`. Optional `underlying_symbol` and `underlying_exchange` override underlying resolution for that item. Those four keys are the whole item schema; anything else returns HTTP 400 for the entire request.

This endpoint uses the shared `API_RATE_LIMIT`, not the `GREEKS_RATE_LIMIT` that the single-symbol [Option Greeks](./optiongreeks.md) endpoint uses.

## Response

```json
{
  "status": "success",
  "data": [
    {
      "status": "success",
      "symbol": "NIFTY30JUL2625000CE",
      "exchange": "NFO",
      "implied_volatility": 15.25,
      "greeks": {
        "delta": 0.52,
        "gamma": 0.0001,
        "theta": -4.97,
        "vega": 30.76,
        "rho": 0.001
      }
    }
  ],
  "summary": {"total": 2, "success": 1, "failed": 1}
}
```

Individual items can fail while the batch response remains successful; inspect every item and the summary. Expired options are normalized to an expired-option Greeks response instead of failing the entire request.

**Back to**: [API documentation](../README.md)
