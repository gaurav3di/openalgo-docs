# Strategy Management

OpenAlgo's Strategy Management Module allows you to automate your trading strategies using webhooks. This enables seamless integration with any platform or custom system that can send HTTP requests. The Strategy class provides a simple interface to send signals that trigger orders based on your strategy configuration in OpenAlgo.



> Note: When creating a trading strategy in the _Strategy Management_ section, ensure you select the platform as Python to access the webhook ID displayed in the _View Strategy_ section.<br>
>
> Keep the webhook ID private, as it is sensitive information similar to API keys. Do not share it with anyone.<br>

`Strategy(host_url, webhook_id)` posts to `{host_url}/strategy/webhook/{webhook_id}`. The webhook accepts JSON with `symbol` and `action`; `position_size` is required as well when the strategy is configured in BOTH mode. The strategy sends orders only for symbols you have mapped in the _View Strategy_ screen, and each mapping carries its own exchange, product type and quantity, so the Python side never sends those.

Trading modes are configured in OpenAlgo, not in the Python code:

* **LONG**: BUY opens, SELL closes.
* **SHORT**: SELL opens, BUY closes.
* **BOTH**: `position_size` is the target position after the order. A non-zero size opens or reverses, `0` squares off. The sign must agree with the action: a BUY needs `position_size >= 0` and a SELL needs `position_size <= 0`, so a short entry of 10 is sent as `-10`. The server rejects a mismatched sign.

Requests to the webhook are rate limited by `WEBHOOK_RATE_LIMIT` (100 per minute by default). Inactive strategies are rejected, and for intraday strategies entries are refused outside the configured start and end time while exits are refused after the square off time.

```python

from openalgo import Strategy

# Initialize strategy client
client = Strategy(
    host_url="http://127.0.0.1:5000",  # Your OpenAlgo server URL
    webhook_id="your-webhook-id"        # Get this from OpenAlgo strategy section
)


# Example 1: LONG or SHORT only mode (configured in OpenAlgo)
# No position_size is sent; the mapped quantity is used
client.strategyorder("RELIANCE", "BUY")
client.strategyorder("RELIANCE", "SELL")


# Example 2: BOTH mode (configured in OpenAlgo)
# position_size is the TARGET position after the order

#Trading Mode - BOTH - Long Entry
client.strategyorder("RELIANCE", "BUY", 10)

#Trading Mode - BOTH - Long Exit
client.strategyorder("RELIANCE", "SELL", 0)

#Trading Mode - BOTH - Short Entry (negative target position)
client.strategyorder("RELIANCE", "SELL", -10)

#Trading Mode - BOTH - Short Exit
client.strategyorder("RELIANCE", "BUY", 0)

```

The client keeps a pooled HTTP connection open, so close it when your script exits, or use it as a context manager:

```python
with Strategy(host_url="http://127.0.0.1:5000", webhook_id="your-webhook-id") as client:
    client.strategyorder("RELIANCE", "BUY")
```
