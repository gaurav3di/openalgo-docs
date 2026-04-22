# Python Strategy Hosting

***

## Hosting Python Strategies in OpenAlgo

OpenAlgo allows you to host and run multiple Python trading strategies as isolated subprocesses. Each strategy operates with its own memory and log files, ensuring that one script does not interfere with another.

{% embed url="https://www.youtube.com/watch?v=pFy2Yx-kqkI" %}

### 1. Accessing the Python Strategy Manager

To begin, log into your OpenAlgo dashboard:

* Click on your **Profile** icon in the top right corner.
* Select **Python Strategies** from the dropdown menu.
* This dashboard provides an overview of your total, running, and scheduled strategies.

### 2. Adding a New Strategy

Click the **Add Strategy** button to open the configuration interface.

#### Uploading the Script

* **Strategy Name:** Enter a unique descriptive name for your strategy.
* **Python Script:** Upload your `.py` file. OpenAlgo supports standard Python libraries and the OpenAlgo SDK.

#### Configuration & Scheduling

* **Exchange:** Select the target exchange (e.g., NSE Equity, MCX Commodity, Crypto). OpenAlgo is holiday-aware and will only run the strategy during valid market hours for the selected exchange.
* **Schedule:** Define the **Start Time** and **Stop Time**. You can also select specific days of the week (Monday–Sunday) to enable or disable the strategy.

### 3. Monitoring & Management

Once a strategy is added, it will appear on your Python Strategies page.

* **Start/Stop:** Use the manual controls to trigger or halt the script execution.
* **View Logs:** Click "View Logs" to see real-time output. This section tracks data fetching, signal generation, and order execution. You can copy or download these logs for debugging.
* **Edit Code:** The built-in code editor allows for quick modifications directly within the browser, though it is recommended to test major changes locally first.

### 4. Execution Modes

OpenAlgo features a toggle to prevent accidental trades during development:

* **Analyzer Mode:** Use this for testing purposes. It allows the script to run and generate logs without sending real orders to the broker.
* **Live Mode:** Once your strategy is verified, switch to Live Mode to enable actual order placement.

### 5. Technical Environment Variables

When your script runs within OpenAlgo, several environment variables are automatically injected. You should use these instead of hardcoding sensitive data:

| Variable           | Description                                             |
| ------------------ | ------------------------------------------------------- |
| `OPENALGO_API_KEY` | Your decrypted API key for the platform.                |
| `STRATEGY_ID`      | A unique identifier for the specific strategy instance. |
| `OPENALGO_HOST`    | The internal host URL for API requests.                 |
| `OPENALGO_WS_URL`  | The WebSocket URL for real-time data streaming.         |

### 6. Verifying Orders

After a trade is triggered by your Python strategy, you can verify it in two places:

1. **Strategy Logs:** Shows the logic and parameters that triggered the trade.
2. **Order Book:** Shows the status (Complete, Rejected, etc.) and details (Price, Quantity, Product Type) of the order sent to the broker.

***

#### Best Practices

* Always test new strategies in **Analyzer Mode** for at least one full trading session.
* Use the **Python Strategy Guide** link within the app for updated code snippets and recommended patterns.
* Ensure your OpenAlgo instance has a stable internet connection if running on a local machine to prevent WebSocket disconnections.
