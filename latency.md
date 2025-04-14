# Latency

## Understanding Latency and Roundtrip in Algo Trading

Latency and roundtrip time (RTT) are crucial concepts in algorithmic trading, especially for traders who aim to optimize their systems for better performance. However, understanding these concepts and their impact on trading strategies is crucial. Let’s break it down and discuss why latency matters, who should care about it, and how trading infrastructure plays a role in your long-term success.

<figure><img src=".gitbook/assets/Screenshot 2025-01-04 at 6.01.17 PM.png" alt=""><figcaption></figcaption></figure>

### What Is Latency?



In trading, latency refers to the time it takes for your system to send an order to the broker’s server and for the broker to respond. It’s measured in milliseconds (ms) and directly affects the speed of order execution.\


**For example:**

• If your one-way latency is 50ms, your system takes 50ms to send the order to the broker’s server.

• Roundtrip time (RTT) includes the time for the broker to process the order and send a response back. If the RTT is 150ms, the one-way latency is around 75ms.

<figure><img src=".gitbook/assets/Screenshot 2025-01-04 at 6.04.05 PM.png" alt=""><figcaption></figcaption></figure>

### What Affects Latency?



Several factors contribute to the latency of an algo trading system:

**1. Proximity to Exchange Servers**

Physical distance plays a significant role. Servers located closer to the exchange experience lower latency due to shorter data transmission distances. For example, hosting a server in Mumbai for Indian markets can reduce latency significantly compared to running the system from Bangalore.

**2. Broker Latency**

This is the time taken by the broker’s infrastructure to process your order and respond. Broker latencies typically range between 30-80ms but can vary depending on the broker’s technology stack and load.

**3. Network Latency**

The quality of your internet connection and the routing of data packets can add delays. For example, a direct connection between your system and the broker’s server will be faster than one routed through intermediate servers.

**4. Application Processing**

The framework or technology stack you use to build your trading application also contributes. Flask, for instance, is moderately fast, while FastAPI can handle more concurrent requests and may reduce processing time slightly.

**5. Use of Tools like Ngrok**

Tools like ngrok add significant latency because they route your data through external servers before reaching the destination. This can increase RTT to 500-700ms, making it unsuitable for low-latency strategies but fine for development, testing, and latency-insensitive trading systems.

<figure><img src=".gitbook/assets/Screenshot 2025-01-04 at 6.08.52 PM.png" alt=""><figcaption></figcaption></figure>

### Why Latency Matters in Algo Trading



The importance of latency depends on the type of trading strategy you are running:



**1. High-Frequency Trading (HFT):**



In HFT, latency is everything. These strategies involve executing thousands of trades in milliseconds to capture small price movements. A difference of even a few microseconds can mean losing out on profitable opportunities to faster competitors.



**2. Medium-Frequency Trading (MFT) and Low-Frequency Trading (LFT):**



For MFT or LFT strategies, latency is far less critical. These strategies execute trades over longer timeframes—seconds, minutes, or even hours. A difference of 100-200ms typically has no significant impact on performance. Most retail traders and algo developers fall into this category, so there’s no need to obsess over ultra-low latency.



**3. Scalping and Arbitrage Strategies:**\


While not as latency-sensitive as HFT, these strategies do benefit from reasonably low latency (e.g., under 200ms). Delays in execution can erode the narrow profit margins these strategies rely on.



### **Real-World Test Results: Vultr Mumbai Server**



To demonstrate the impact of server proximity, let’s compare two setups:

**1. Vultr Mumbai Data Center (2 cores, 4GB RAM)**

• Average Roundtrip Time (RTT): \~144ms

• Latency (One-Way): \~72ms

<figure><img src=".gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

**2. Local Machine in Bangalore**

• RTT: \~166-222ms

• Latency (One-Way): \~83-111ms



The Mumbai server, being closer to the exchange, has significantly lower latency than the Bangalore setup. This shows how proximity to the exchange can reduce latency and improve execution speed.

<figure><img src=".gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

\


This includes:

&#x31;**. Broker Latency**: Typically 30-80ms for AngelOne.

&#x32;**. Network Delay**: Approximately 50-70ms due to the distance from Bangalore to Mumbai.

**3. Processing Overheads**: Another 30-50ms for application and server processing.\


These values are normal and expected for algo trading systems operating from a non-co-located setup.\


**Why Trading Infrastructure Matters in the Long Run**\


While latency may not seem critical for most retail or MFT strategies, investing in a solid trading infrastructure pays off over time. Here’s why:

**1. Reliability:**

Downtime during volatile market conditions can be costly. A robust infrastructure ensures consistent uptime and smooth execution.

**2. Scalability:**

As your strategies grow in complexity or volume, your infrastructure should be able to handle the load without slowing down.

**3. Reducing Slippage:**

Even for MFT strategies, lower latency helps reduce slippage, which can add up over time and affect overall profitability.



### When to Ignore Latency



Latency isn’t always the most critical factor. For most retail traders running MFT or LFT strategies, the focus should be on:

• Robust strategy logic

• Execution reliability

• Minimizing errors and downtime



In these cases, the difference between 100ms and 200ms latency will have a negligible impact on performance.

### Takeaways for Budding Algo Traders

**1. Latency isn’t everything**: For MFT or LFT strategies, focus on building reliable, consistent systems. Latency only becomes critical for HFT or scalping/arbitrage strategies.

**2. Server location matters:** Hosting servers close to the exchange (e.g., Vultr Mumbai for Indian markets) can significantly reduce latency.

**3. Ngrok is not for latency sensitive strategy**: While ngrok is great for development and testing, it’s not ideal for live trading systems that require low latency. If you are using NGROK for connecting your tradingview/chartink aware about the 500-800 ms of latency it could create if you are running in your local desktop.

**4. Plan for growth**: Investing in scalable infrastructure ensures your system can handle increased complexity or volume as you evolve.\


In conclusion, while latency matters for specific strategies, most traders should prioritize strategy robustness, execution consistency, and scalability over chasing the lowest possible latency. A solid understanding of latency and infrastructure will prepare you for success in both the short and long term.

