# Chrome Extension

A lightweight Chrome extension for the OpenAlgo trading platform with a DaisyUI-inspired modern UI.



{% embed url="https://www.youtube.com/watch?v=EqPyfyMA8lM" %}

### Features

* **Quick Trading Actions**: Instantly place trades using LE (Long Entry), LX (Long Exit), SE (Short Entry), and SX (Short Exit) buttons.
* **Minimal Interface**: Compact, draggable layout that integrates directly onto chart pages.
* **Integrated Settings Panel**: Easy configuration using a settings popup with support for multiple exchanges and trading products.
* **Modern UI**: Styled using DaisyUI for a clean and efficient interface.
* **Real-time Feedback**: Visual confirmation for each trading action.



### Download OpenAlgo Chrome Extension

{% embed url="https://github.com/marketcalls/openalgo-chrome/releases/tag/v1.0" %}

### Prerequisites

* **OpenAlgo API Server**: The OpenAlgo server must be running (locally or remotely) and accessible.
* **API Key**: A valid API key is required to authenticate API requests.
* **Browser**: Chrome version 88 or higher. (Recommended Latest Version of Chrome)

### Installation Guide

#### Install via Developer Mode

1. Open Chrome and go to `chrome://extensions/`
2. Enable **Developer mode** (top right toggle)
3. Click on **"Load unpacked"**
4. Select the folder you extracted from the `.zip` file
5. The OpenAlgo extension icon will now appear in your Chrome toolbar

<figure><img src="../.gitbook/assets/image (93).png" alt=""><figcaption></figcaption></figure>

### Usage Guide

#### Getting Started

1. **Start the OpenAlgo Server**: Make sure the OpenAlgo server is running (default: `http://127.0.0.1:5000`)

<figure><img src="../.gitbook/assets/image (92).png" alt=""><figcaption></figcaption></figure>

1. **Configure Settings**:
   * Click the 3-dot (⋮) icon next to the button panel
   * Fill in the following fields:
     * Host URL (e.g., `http://127.0.0.1:5000`)
     * API Key
     * Symbol (e.g., `RELIANCE` or `NIFTY27APR25FUT`)
     * Exchange (e.g., NSE, BSE, NFO, etc.)
     * Product (MIS, NRML, CNC)
     * Quantity (e.g., 10, 20, etc.). Futures or Options symbols it is always the total number of shares instead of Lot Size
2. **Save Settings**: Click **Save** to apply your configuration

<figure><img src="../.gitbook/assets/image (91).png" alt=""><figcaption></figcaption></figure>

#### Trading Actions

* **LE (Green)**: Long Entry (Buy to Open)
* **LX (Yellow)**: Long Exit (Sell to Close Long)
* **SE (Red)**: Short Entry (Sell to Open Short)
* **SX (Blue)**: Short Exit (Buy to Close Short)

These buttons appear on supported charting pages and can be repositioned.

#### Draggable Interface

* Hover over the top edge of the trading bar to reveal the grab handle
* Click and drag to reposition the widget anywhere on the screen

### Troubleshooting

* **No Response from Buttons**:
  * Ensure the OpenAlgo server is running
  * Double-check Host URL and API Key
* **Incorrect or Missing Settings**:
  * Reopen settings panel and verify all fields are filled
* **Server Connection Errors**:
  * Confirm the server is reachable at the provided Host URL

***

The OpenAlgo Chrome Extension is designed to simplify and streamline your trading experience. With a clean interface and real-time trading controls built right into your browser, it allows you to react quickly to market opportunities with just one click. Whether you're managing long or short positions, this extension brings the power of OpenAlgo directly into your charting tools, making intraday trading more efficient and intuitive. Configure once, trade instantly—let technology work for you while you focus on strategy.
