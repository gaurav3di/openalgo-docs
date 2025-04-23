---
description: OpenAlgo - Model Context Protocol
---

# MCP

1. **User:** Interacts via the terminal with the MCP Client.
2. **User Prompt:** Types a command in natural language (e.g., "Get my positions").
3. **MCP Client (Trading Agent):**
   * Uses Agno AGI and an LLM to parse the prompt and determine the intent (e.g., `get_position_book` tool).
   * Extracts parameters if needed.
   * Sends a structured MCP request to the MCP Server.
4. **MCP Server:**
   * Receives the MCP request.
   * Calls the corresponding OpenAlgo API endpoint (e.g., `/positions`).
   * Receives the response from OpenAlgo.
   * Formats the response according to MCP and sends it back to the MCP Client.
5. **MCP Client (Trading Agent):**
   * Receives the MCP response.
   * Optionally uses the LLM again to format the response naturally for the user.
   * Displays the result to the User.
6. **OpenAlgo:** The core OpenAlgo application handles the actual API logic and communication with the broker.
7. **Broker/Exchange:** Executes the trade or provides the requested data.

<figure><img src=".gitbook/assets/image (106).png" alt=""><figcaption></figcaption></figure>

### Core Technologies

* **OpenAlgo:** The underlying trading platform providing the core API.
* **Model Context Protocol (MCP):** Communication standard for AI agents and tools.
* **Agno AGI:** Agent framework for building the trading assistant client.
* **Python:** Primary programming language.
* **FastAPI/Starlette:** Web framework for the MCP Server.
* **LLM APIs:** Connectors for various Large Language Models (e.g., OpenAI Python library).

### Available Tools / Capabilities (Examples)

The Trading Agent can typically perform actions corresponding to OpenAlgo API endpoints, such as:

* `get_funds()`: Retrieve available cash and margin details.
* `get_holdings()`: Get list of holdings.
* `get_position_book()`: Get current open positions.
* `get_order_book()`: Get status of today's orders.
* `get_trade_book()`: Get list of executed trades for the day.
* `get_quote(symbol, exchange)`: Get LTP, bid/ask for a symbol. Supports multiple symbols.
* `get_history(symbol, exchange, interval, start_date, end_date)`: Get historical candle data.
* `place_order(symbol, exchange, quantity, transaction_type, order_type, product_type, price, trigger_price)`: Place a buy or sell order.
* `place_basket_order(orders)`: Place multiple orders simultaneously.
* `modify_order(order_id, ...)`: Modify a pending order.
* `cancel_order(order_id)`: Cancel a pending order.
* `get_symbol_metadata(...)`
* `get_all_tickers()`

_(Refer to the specific implementation in `server/server.py` or similar within the repository for the exact list and parameters)._

### Setup and Installation

1. **Prerequisites:**
   * Python 3.8+ installed.
   * Git installed.
   * A running instance of the core OpenAlgo application.
2.  **Clone the Repository:**

    ```bash
    git clone https://github.com/marketcalls/openalgo-mcp.git
    cd openalgo-mcp
    ```
3.  **Set up Virtual Environment (Recommended):**

    ```bash
    python -m venv venv
    # Activate the environment
    # Windows:
    .\venv\Scripts\activate
    # macOS/Linux:
    source venv/bin/activate
    ```
4. **Install Dependencies:**
   *   Install requirements for both server and client:

       ```bash
       pip install -r server/requirements.txt
       pip install -r client/requirements.txt
       ```
   * _(Note: Check the repository structure; requirements might be combined or located differently)_
5. **Configure Environment Variables:**
   * Create a `.env` file in the root directory of the cloned project (or potentially separate ones in `server` and `client` directories if structured that way).
   *   Add the following required variables:

       ```dotenv
       # .env file

       # For the MCP Client (Agno AGI) - Connects to the LLM
       OPENAI_API_KEY="sk-YourOpenAISecretKey" # Replace with your actual key
       # Add other LLM keys if using different providers (e.g., GOOGLE_API_KEY)

       # For the MCP Server - Connects to your OpenAlgo Instance
       OPENALGO_API_HOST="http://127.0.0.1:5000" # Or your OpenAlgo host URL
       OPENALGO_API_KEY="YourOpenAlgoApiKey"     # Replace with your OpenAlgo API Key
       ```
   * _Ensure the `OPENALGO_API_KEY` has the necessary permissions (order placement, data retrieval) enabled in your core OpenAlgo settings._

### Running the Application

You need to run **three** separate processes, ideally in different terminal windows:

1. **Start OpenAlgo Core Application:**
   * Navigate to your main OpenAlgo installation directory.
   * Start it as you normally would (e.g., `python app.py`).
   * Ensure it's running and accessible at the `OPENALGO_API_HOST` specified in the `.env` file.
2. **Start the MCP Server:**
   * Open a new terminal.
   * Navigate to the `openalgo-mcp/server` directory (or wherever `server.py` is).
   * Activate the virtual environment (`source venv/bin/activate` or `.\venv\Scripts\activate`).
   *   Run the server:

       ```bash
       python server.py
       # Or potentially: uvicorn server:app --reload --port 8001 (Check repository instructions)
       ```
   * Keep this terminal open. It usually runs on a port like `8001`.
3. **Start the MCP Client (Trading Agent):**
   * Open a _third_ terminal.
   * Navigate to the `openalgo-mcp/client` directory (or wherever `trading_agent.py` is).
   * Activate the virtual environment (`source venv/bin/activate` or `.\venv\Scripts\activate`).
   *   Run the client:

       ```bash
       python trading_agent.py
       ```
   * This terminal will now show the chat prompt `Enter your query: (or 'quit' to exit):`.

### Usage Examples

Once the client is running, you can type natural language commands:

* `Hi who are you`
* `get the funds details`
* `get the quotes of SBIN`
* `get the quotes of SBIN & TCS, RELIANCE in table format`
* `place order in YESBANK 10 shares in MIS`
* `buy 10 shares in NHPC and 5 shares in ZEEL in MIS`
* `get the positionbook in table format`
* `show me my orders`
* `cancel order <order_id>`

Type `quit` to exit the client application.

### Inspiration and Acknowledgements

* This project was inspired by the work done by **Amit Wani** on the [zerodha-mcp](https://github.com/mtwn105/zerodha-mcp) project, which pioneered the MCP concept for Zerodha Kiteconnect.
* Thanks to the **OpenAlgo** team for the core trading platform.
* Thanks to the developers of the **Agno AGI** framework.

### License

This project is licensed under the **Apache 2.0 License**. See the LICENSE file for details.
