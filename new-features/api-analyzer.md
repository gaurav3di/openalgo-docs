# API Analyzer

### What is Sandbox Mode?

**Sandbox Mode** (also called **API Analyzer Mode**) is like a **trading video game** where you can practice trading strategies with **fake money** but **real market prices**.

Think of it as:

* **Flight Simulator** for trading
* **Real market prices** from live data
* **Safe testing** - no real money at risk

### Why Would You Use It?

#### Real-World Examples

**Example 1: Learning Algorithmic Trading**

* You wrote your first Python trading bot
* You're scared to test it with real money
* Enable Sandbox → Get ₹1 Crore fake money → Test your bot safely
* If it loses all the money, no problem! Just reset and try again

**Example 2: Testing a New Strategy**

* You read about a new momentum trading strategy
* Before risking ₹1 lakh real money, test it in sandbox
* Run it for a week with fake money
* If it works well, switch to live mode

**Example 3: Practicing Manual Trading**

* You're new to options trading
* You want to practice buying/selling options without losing money
* Sandbox gives you unlimited practice rounds

**Example 4: Debugging Your Code**

* Your trading bot has a bug that's placing wrong orders
* Instead of debugging with real money (expensive!), use sandbox
* Fix all bugs safely, then go live

### How It Works

#### The Magic Setup

When you enable Sandbox Mode:

1. OpenAlgo provides virtual simulation of a portfolio
2. All your orders execute in a **simulated environment**
3. Prices come from **real-time market data**
4. **Everything else works exactly like live trading**
5. **Nothing touches your real broker account**

#### Live Mode vs Sandbox Mode

| Feature             | Live Mode        | Sandbox Mode       |
| ------------------- | ---------------- | ------------------ |
| **Money**           | Real ₹₹₹         | Dummy Capital      |
| **Prices**          | Real market      | Real market (same) |
| **Orders**          | Go to broker     | Stay in OpenAlgo   |
| **P\&L**            | Real profit/loss | Simulated P\&L     |
| **Risk**            | HIGH             | ZERO               |
| **Can lose money?** | Yes              | No                 |
| **Good for**        | Making money     | Learning, testing  |

### Step-by-Step Setup

#### Step 1: Enable Sandbox Mode

**Method A: Using Web UI** (Easiest)

1. Log in to OpenAlgo
2. Click **Settings** in the menu
3. Find **"API Analyzer Mode"** toggle
4. Switch it **ON** (it will turn green/blue)
5. Notice: Interface changes to a different theme (visual indicator)

**Method B: Using API** (For Python users)

```python
import requests

payload = {
    "apikey": "YOUR_API_KEY",
    "mode": True  # True = Sandbox ON, False = Sandbox OFF
}

response = requests.post(
    "http://127.0.0.1:5000/api/v1/analyzer/toggle",
    json=payload
)
print(response.json())
```

#### Step 2: Check Your Fake Money

Go to **Funds** page:

* Available Balance: ₹10,000,000&#x20;
* Used Margin: ₹0
* Total P\&L: ₹0

#### Step 3: Place Your First Test Order

**Using Web UI:**

1. Go to any trading page
2. Enter symbol (e.g., "RELIANCE")
3. Choose BUY/SELL
4. Enter quantity (e.g., 10 shares)
5. Click "Place Order"
6. Order executes instantly with fake money!

**Using Python Code:**

```python
import requests

payload = {
    "apikey": "YOUR_API_KEY",
    "symbol": "RELIANCE",
    "exchange": "NSE",
    "action": "BUY",
    "quantity": 10,
    "pricetype": "MARKET",
    "product": "MIS"
}

response = requests.post(
    "http://127.0.0.1:5000/api/v1/placeorder",
    json=payload
)
print(response.json())
```

#### Step 4: Watch It Work!

1. Check **Order Book** - Your order appears as "Complete"
2. Check **Positions** - Shows your position with P\&L
3. Check **Funds** - Used margin deducted from available balance
4. P\&L updates in real-time as market price changes!

### What You Get in Sandbox

#### 1. Starting Capital

* **Default**: ₹1 Crore (₹10,000,000)
* **Configurable**: You can change it in sandbox settings
* **Auto-reset**: Resets to starting amount every Sunday (configurable)

#### 2. All Order Types Work

* **MARKET** - Buy/sell at current market price
* **LIMIT** - Buy/sell at specific price
* **SL (Stop Loss)** - Trigger at stop price
* **SL-M (Stop Loss Market)** - Trigger and execute at market

#### 3. All Product Types Work

* **MIS (Intraday)** - Squared off automatically at 3:15 PM
* **CNC (Delivery)** - Converted to holdings at midnight
* **NRML (F\&O)** - Futures and options trading

#### 4. Realistic Features

**Margin System:**

* Equity Intraday (MIS): 5x leverage
* Equity Delivery (CNC): 1x leverage (full payment)
* Futures: 10x leverage
* Options Buy: 1x (pay full premium)
* Options Sell: 10x leverage

**Auto Square-Off:**

* MIS positions close automatically at:
  * NSE/BSE: 3:15 PM
  * MCX: 11:30 PM
  * CDS: 4:45 PM

**Realistic Pricing:**

* BUY orders execute at **ask price** (seller's price)
* SELL orders execute at **bid price** (buyer's price)
* Just like real market!

### Sandbox Settings Page

Navigate to: `http://127.0.0.1:5000/sandbox`

#### What You Can Configure

**1. Capital Settings**

* Starting Capital: ₹10,000,000 (change it to any amount)
* Auto-Reset Day: Sunday (pick any day)
* Auto-Reset Time: 12:00 AM (pick any time)
* Manual Reset: Click "Reset Now" button anytime

**2. Leverage Settings**

* Equity MIS: 5x (how much you can borrow for intraday)
* Futures: 10x (margin required for futures)
* Options Sell: 10x (margin for selling options)
* Options Buy: 1x (must pay full premium)

**3. Square-Off Times**

* When MIS positions auto-close
* Different times for NSE, MCX, CDS

**4. Performance Tuning**

* How often to check pending orders (5 seconds)
* How often to update P\&L (5 seconds)

### How Sandbox Handles Different Scenarios

#### Scenario 1: Buying Stocks (CNC)

**What you do:**

```
BUY 10 RELIANCE @ ₹2,500 (CNC)
```

**What happens:**

1. **Order placed** → ₹25,000 blocked from available balance
2. **Position created** → Shows in positions page
3. **Next day midnight** → Position converts to holdings automatically
4. **Holdings page** → Shows 10 RELIANCE shares

#### Scenario 2: Intraday Trading (MIS)

**What you do:**

```
BUY 100 SBIN @ ₹625 (MIS)
Price goes up to ₹630
```

**What happens:**

1. **Margin blocked** → ₹12,500 (₹62,500 / 5x leverage)
2. **Position shows** → +₹500 profit (100 shares × ₹5)
3. **At 3:15 PM** → Position automatically squared off
4. **Profit credited** → Available balance increases by ₹500

#### Scenario 3: Options Trading

**What you do:**

```
BUY NIFTY 20000 CE @ ₹100 (1 lot = 50 qty)
Total cost: ₹5,000
```

**What happens:**

1. **₹5,000 blocked** (full premium)
2. **If premium goes to ₹150** → Profit ₹2,500
3. **If premium goes to ₹50** → Loss ₹2,500
4. **Just like real options trading!**

#### Scenario 4: Running Out of Money

**What happens:**

```
Available Balance: ₹10,000
You try to BUY ₹50,000 worth of stock
```

**Result:**

* Order rejected
* Error: "Insufficient funds"
* Just like real trading!

### Understanding P\&L (Profit & Loss)

#### Unrealized P\&L

* Profit/loss on **open positions**
* Changes every second as market price changes
* Not real until you close the position

#### Realized P\&L

* Profit/loss on **closed positions**
* Added to your available balance
* This is "real" profit/loss in sandbox

#### Example

```
1. BUY 100 SBIN @ ₹625
2. Current price: ₹630
   Unrealized P&L: +₹500 (not closed yet)
3. SELL 100 SBIN @ ₹630
   Realized P&L: +₹500 (closed, profit credited)
```

### Common Questions

#### Q: Is this the same as "paper trading"?

**A:** It's similar but better! It's a **self-hosted testing environment** that uses real market data. It's NOT a regulated paper trading platform (which SEBI doesn't allow in India).

#### Q: Can I lose real money in sandbox?

**A:** **NO!** Absolutely not. All money in sandbox is fake. Your real broker account is never touched.

#### Q: Does my broker know I'm using sandbox?

**A:** **NO!** Sandbox runs entirely within OpenAlgo. No orders go to your broker.

#### Q: Can I test my algo trading strategies?

**A:** **YES!** That's the main purpose. Test all your strategies safely before going live.

#### Q: What happens when I switch back to live mode?

**A:** All sandbox data stays in sandbox database. Your live trading starts fresh. They're completely separate.

#### Q: Can I reset my sandbox funds?

**A:** **YES!** Two ways:

1. **Auto-reset**: Every Sunday at midnight (configurable)
2. **Manual reset**: Click "Reset Now" button in sandbox settings

#### Q: Do I need real broker API keys to use sandbox?

**A:** **NO!** Sandbox works without broker connection. It simulates everything. However, if you have broker connection, it uses real-time prices (better realism).

#### Q: Can I use Action Center with Sandbox?

**A:** **YES!** They work together:

* Enable both Sandbox + Semi-Auto mode
* Test orders → Go to Action Center → You approve/reject
* Perfect for testing the full workflow

### When to Use Sandbox vs Live

#### Use Sandbox Mode When:

* Testing new strategies
* Learning algorithmic trading
* Debugging your trading code
* Practicing options/futures trading
* Experimenting with order types
* Validating your algorithm logic
* You're not ready to risk real money

#### Use Live Mode When:

* Strategy tested and proven in sandbox
* You understand the risks
* Code is bug-free
* You're ready to make real money (or lose it)
* You've practiced enough

### Best Practices

#### 1. Always Test First

```
New Strategy → Sandbox (1 week) → If profitable → Live Mode
```

#### 2. Use Realistic Capital

If you plan to trade live with ₹1 lakh, test sandbox with ₹1 lakh (not ₹1 crore). This gives realistic results.

#### 3. Monitor Performance

* Track win rate
* Calculate average profit per trade
* Measure maximum drawdown
* Use this data to improve strategy

#### 4. Don't Skip Sandbox

Even experienced traders test new strategies in sandbox first. It's free insurance!

#### 5. Combine with Action Center

* Sandbox ON + Semi-Auto ON = Perfect testing environment
* See every order before execution
* Catch bugs early

### Visual Indicators

When sandbox is enabled:

* **Theme changes** to "Garden" theme (different colors)
* **Badge/label** shows "Analyzer Mode Active"
* **Funds page** shows sandbox balance (₹1 Crore default)
* **Separate database** (sandbox.db)

When live mode is active:

* **Default theme**
* **Real broker funds** shown
* **Real database** (openalgo.db)

### Technical Details (Optional)

#### Database Separation

* **Live**: `db/openalgo.db`
* **Sandbox**: `db/sandbox.db`
* Completely isolated - no data mixing

#### Architecture

```
Your Trading Bot
      ↓
OpenAlgo API
      ↓
  [Analyzer Check]
      ↓
   Sandbox?
   ↙     ↘
YES        NO
 ↓          ↓
Sandbox    Real Broker
Engine     API
```

#### Thread Management

* Sandbox runs in background thread
* Monitors pending orders every 5 seconds
* Updates P\&L in real-time
* Auto square-off runs on schedule
* T+1 settlement at midnight

#### Configuration File

Located at: `db/sandbox.db` in table `sandbox_config`

Can be modified via sandbox settings page or directly in database.

### Troubleshooting

#### Problem: Orders not executing in sandbox

**Solution:**

1. Check if sandbox mode is ON (Settings → API Analyzer Mode)
2. Verify symbol exists (try RELIANCE, SBIN, INFY)
3. Check available balance (Funds page)
4. Look at logs for errors

#### Problem: Can't see positions

**Solution:**

1. Make sure you're looking at sandbox positions, not live
2. Check if order was actually placed (Order Book)
3. Verify order status is "Complete"

#### Problem: Wrong P\&L calculation

**Solution:**

1. Remember: BUY @ ask price, SELL @ bid price
2. Check if using correct product type (MIS vs CNC)
3. Verify leverage settings in sandbox config

#### Problem: Funds not resetting

**Solution:**

1. Check auto-reset day/time in sandbox settings
2. Use manual "Reset Now" button
3. Restart OpenAlgo application

### Example: Complete Trading Workflow

Let's walk through a complete example:

#### Step 1: Enable Sandbox

```
Settings → API Analyzer Mode → ON
```

#### Step 2: Check Starting Balance

```
Funds page: ₹10,000,000
```

#### Step 3: Place First Order

```python
# Buy RELIANCE
{
    "symbol": "RELIANCE",
    "action": "BUY",
    "quantity": 10,
    "pricetype": "MARKET",
    "product": "MIS"
}
```

#### Step 4: Check Position

```
Positions page:
- Symbol: RELIANCE
- Quantity: 10
- Avg Price: ₹2,500
- LTP: ₹2,505
- P&L: +₹50
```

#### Step 5: Close Position

```python
# Sell RELIANCE
{
    "symbol": "RELIANCE",
    "action": "SELL",
    "quantity": 10,
    "pricetype": "MARKET",
    "product": "MIS"
}
```

#### Step 6: Check Final P\&L

```
Trade Book:
- BUY 10 @ ₹2,500 = -₹25,000
- SELL 10 @ ₹2,505 = +₹25,050
- Realized P&L: +₹50

Funds:
- Available Balance: ₹10,000,050
- Profit: ₹50 ✅
```

#### Step 7: Analyze and Learn

* Strategy worked!
* Made ₹50 profit (in sandbox)
* Now optimize and test more
* When confident → Switch to Live Mode

### Regulatory Note

#### Is Sandbox Legal in India?

**YES**

Sandbox is **NOT** a virtual/paper trading platform.

It's a **personal testing environment** that:

* Runs on your own computer
* Uses your own OpenAlgo installation
* For your own strategy development
* Like a calculator or Excel spreadsheet

It's a **development tool**, not a trading service.

### Support & Resources

* **Detailed Docs**: See `docs/sandbox/` folder for technical details
* **Report Issues**: https://github.com/marketcalls/openalgo/issues
* **Full Documentation**: https://docs.openalgo.in
* **Community**: Join discussions on GitHub

### Summary

**Sandbox Mode** = Your **safe playground** for trading

* Free virtual money (₹1 Crore)
* Real market prices
* Zero risk
* Perfect for learning
* Test before going live

**Remember**: The best traders test everything before risking real money. Sandbox makes that easy!

***

**Next Steps**:

1. Enable sandbox mode
2. Place a few test orders
3. Watch how P\&L changes
4. Test your strategy for 1 week
5. If profitable, switch to Live Mode
6. Start making real money

Good luck and happy testing!
