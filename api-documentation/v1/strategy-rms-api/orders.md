# Order History

Read the strategy engine's durable order records across all owned runs, or for one run.

```http
POST /api/v1/strategy/orders
Content-Type: application/json
```

```json
{
  "apikey": "<your_app_apikey>",
  "strategy_id": 7,
  "run_id": 42
}
```

`run_id` is optional and must be a positive integer. A run that belongs to another strategy returns an empty list rather than leaking orders. There is no limit parameter; narrow by run when appropriate.

## Response

```json
{
  "status": "success",
  "data": [
    {
      "id": 318,
      "run_id": 42,
      "leg_id": 1,
      "kind": "entry",
      "position_ref": "969bc536b1c14d15992f730c2c136d7a",
      "broker_order_id": "26083004118201",
      "symbol": "NIFTY04SEP2624500CE",
      "exchange": "NFO",
      "action": "SELL",
      "qty": 75,
      "product": "NRML",
      "pricetype": "MARKET",
      "status": "complete",
      "placed_at": "2026-08-30T03:50:11.610224+00:00",
      "filled_at": "2026-08-30T03:50:12.004881+00:00",
      "avg_fill_price": 142.35,
      "filled_qty": 75,
      "reject_reason": null
    }
  ]
}
```

Rows are oldest first by `placed_at`, so an entry precedes its exit. The row is intentionally created before broker dispatch; `pending` with a null `broker_order_id` can therefore be a real, recoverable intent rather than an absent order.

`kind` is one of `entry`, `exit_sl`, `exit_target`, `exit_trail`, `exit_overall_sl`, `exit_overall_target`, `exit_lock_profit`, `exit_eod`, `exit_expiry`, `exit_daily_loss_limit`, `exit_close_all`, `exit_leg_manual`, `exit_recovery`, or `exit_signal`.

`product` is the value actually sent to the venue. It can differ from the strategy's configured product intent: for example, a carry configuration is sent as `NRML` for a derivatives leg and `CNC` for cash. It can be `null` for records created before that column existed.

A positive `filled_qty` proves exposure even if status is `open`, `cancelled`, or `rejected`, because working orders can partially fill. Missing `avg_fill_price` means valuation is unavailable, not zero. These records state what the strategy asked for; the broker orderbook is the authority for the broker's present order state.

---

**Related**: [Run History](runs.md) | [Risk Event Audit Trail](events.md) | **Back to**: [Strategy RMS API](README.md)
