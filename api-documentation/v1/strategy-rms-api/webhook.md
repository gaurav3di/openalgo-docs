# Public Strategy Webhook

Use this endpoint for TradingView and other alert senders. It is **not** part of the V1 RESTX API and does not accept `apikey`.

```http
POST /strategy/webhook/<token>
Content-Type: application/json
```

The high-entropy URL token is the credential. It is shown once in the browser when a strategy is created or rotated; OpenAlgo stores only its SHA-256 digest. Do not send an API key in the alert body. If the token may have appeared in a sender configuration, proxy log, support bundle, or custom access log, rotate it immediately on the strategy page.

## Batch strategy messages

A batch strategy accepts only `start` and `stop`.

```json
{
  "action": "start",
  "mode": "sandbox"
}
```

```json
{
  "action": "stop"
}
```

`mode` is required for `start` and must be `live` or `sandbox`. It is ignored for `stop`. The webhook trims and lowercases `action` and `mode`, but does not default a missing mode. Live remains refused until the operator enables live trading on the strategy.

## Signal strategy messages

A signal strategy accepts only `long_entry`, `long_exit`, `short_entry`, and `short_exit`. Identify a leg by `leg_id`, or by `symbol` and `exchange`; `leg_id` wins if both are present.

```json
{
  "action": "long_entry",
  "leg_id": 1
}
```

```json
{
  "action": "short_exit",
  "symbol": "RELIANCE",
  "exchange": "NSE"
}
```

There is no `start` or `mode` in signal mode. The first accepted signal after the strategy session boundary opens the run. A repeat entry or exit for a position not held is a 200 no-op such as `Signal accepted (already_long)`; an incompatible direction or leg is a 400 configuration rejection.

## Response

```json
{
  "status": "success",
  "result": "ok",
  "message": "Strategy start accepted",
  "strategy_id": 7,
  "run_id": 42
}
```

A batch stop additionally includes `stop_pending` and `exits`. `stop_pending: true` means the stop is durable but exits still need fill, retry, or reconciliation. It is not proof that the broker account is flat.

| Result | HTTP | Meaning |
|---|---:|---|
| `ok` | 200 | Alert accepted; a signal no-op is also an accepted result. |
| `rejected_token` | 404 | Token is malformed, unknown, or was rotated. |
| `rejected_locked` | 403 | The strategy webhook kill switch is engaged. |
| `rejected_ip` | 403 | The sender address is outside the configured CIDR allowlist. |
| `rejected_payload` | 400 | Body is invalid, not a JSON object, or exceeds the admitted size cap. |
| `rejected_invalid_action` | 400 | Action, batch mode, or strategy-kind configuration is invalid. |
| `rejected_live_disabled` | 403 | A live batch start was not enabled by the operator. |
| `rejected_dedupe` | 200 | An identical batch delivery was already handled within 60 seconds. |
| `rejected_cooling_off` | 409 | A batch start arrived within 30 seconds of a stop. |
| `rejected_engine_error` | 500 | The engine could not accept the request. |
| `rate_limited` | 429 | The route rate limiter refused the request. |

`status` reports HTTP success/error semantics, while `result` records the audit outcome. A deduplicated retry is a 200 because its earlier delivery already satisfied the caller's intent.

## Security And Validation Order

Before the validation pipeline, the route applies its rate limit and rejects a declared oversized body with 413 **before reading the body**. Those two preflight failures do not create a durable webhook audit row. The declared-size cap is 16,384 bytes.

Admitted requests are checked in order: token, kill switch, client IP allowlist, JSON body, action for this strategy kind, batch mode and live gate, duplicate/cooling-off gates where applicable, then engine dispatch. Every admitted terminal result is audited.

The non-empty IP allowlist is a closed CIDR set. OpenAlgo uses the real client address as resolved through proxy rules, not the reverse proxy address. The route has both per-client and per-token limits; the token limit is keyed by a SHA-256 digest, never the raw token.

Application logs, traffic records, and shipped nginx access-log configuration redact webhook token path segments. External alert services and custom TLS proxies are outside that boundary, so they must be configured not to expose the URL credential.

---

**Back to**: [Strategy RMS API](README.md) | [V1 API documentation](../README.md)
