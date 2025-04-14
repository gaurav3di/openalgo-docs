# Strategy Management

OpenAlgo's Strategy Management Module allows you to automate your trading strategies using webhooks. This enables seamless integration with any platform or custom system that can send HTTP requests. The Strategy class provides a simple interface to send signals that trigger orders based on your strategy configuration in OpenAlgo.



> Note: When creating a trading strategy in the _Strategy Management_ section, ensure you select the platform as Python to access the webhook ID displayed in the _View Strategy_ section.\
>
>
> Keep the webhook ID private, as it is sensitive information similar to API keys. Do not share it with anyone.\
>

```python

from openalgo import Strategy

# Initialize strategy client
client = Strategy(
    host_url="http://127.0.0.1:5000",  # Your OpenAlgo server URL
    webhook_id="your-webhook-id"        # Get this from OpenAlgo strategy section
)


# Example 1: Long/Short only mode (configured in OpenAlgo)
#Trading Mode - Long/Short only mode (configured in OpenAlgo)
client.strategyorder("RELIANCE", "BUY")
client.strategyorder("RELIANCE", "SELL")


#Trading Mode - BOTH - Long Entry
client.strategyorder("RELIANCE", "BUY",10)


# Example 2: BOTH mode (configured in OpenAlgo)
#Trading Mode - BOTH - Long Exit
client.strategyorder("RELIANCE", "SELL",0)


#Trading Mode - BOTH - Short Entry
client.strategyorder("RELIANCE", "SELL",10)


#Trading Mode - BOTH - Short Exit
client.strategyorder("RELIANCE", "BUY",0)
 
```
