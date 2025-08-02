# Static IP

### Key FAQs About Static IP Regulation

#### 1. **Can I deploy my algo on AWS or other cloud services without extra cost for IPs?**

No. You’ll need to get and register a static IP even on cloud platforms. This is a mandatory part of the new compliance rules. However VPS providers like Digitalocean, Vultr, OVH by default comes with static IP

***

#### 2. **What if I travel often or work from different locations?**

You can update your registered IP, but only once a week at the Broker API Developer portal. So occasional changes are allowed, but daily switching isn’t feasible.

***

#### 3. **Is it possible to register more than one static IP?**

Yes, you can register both a primary and a backup IP per app. However, changing IPs frequently goes against the brokers guidelines.

***

#### 4. **Do I need a static IP if I’m just streaming market data?**

No. If your app only receives data and doesn’t place or modify orders, you don’t need to register a static IP. Also check whether your broker supports such requirements at the API developer portal. It could be mostly broker dependent.

***

#### 5. **Can I use an IP address from any country?**

Yes, as long as the country is not on the restricted list. You can host from India , US , Europe or other approved regions.

***

#### 6. **What if I’m a service provider placing trades for clients?**

If you're managing trades on behalf of others, you need to be officially registered as a vendor or research analyst. Simply placing orders from a shared IP won’t comply.

***

#### 7. **Will I need to register if my app sends too many orders?**

If your strategy consistently places over 10 orders per second, you may need to go through the formal registration process with the broker. Occasional spikes are okay if your code handles them.

***

#### 8. **Can I use one static IP for multiple trading accounts?**

You can use the same IP across different brokers. But if you’re managing multiple accounts with the same broker, each one may require its own registered IP.

***

