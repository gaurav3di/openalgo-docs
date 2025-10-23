# Amibroker Plugin

## OpenAlgo Amibroker Plugin Setup Guide

This guide will walk you through the installation and configuration of the OpenAlgo Plugin for AmiBroker.

### Prerequisites

* AmiBroker 6.0+ or higher installed on your system
* OpenAlgo server running (ensure it's accessible at your configured host URL)
* Your OpenAlgo API key



{% embed url="https://www.youtube.com/watch?v=MaRM4eZHLJw" %}

### Features

-1-minute intraday data (last 30 days)\
-Daily (EOD) data (up to 1 year)\
-Intelligent backfill with gap detection\
-Incremental updates (fetch only new data)\
-Connects to all OpenAlgo-supported brokers providing historical data and live quotes\
-Multi Exchange Coverage NSE, BSE, MCX, NFO, CDS\
-symbol format INFY-NSE , CRUDEOIL19NOV25FUT-MCX, TCS-BSE (symbol : -OpenAlgo Symbol, Exchange - OpenAlgo Exchange format)\
-Visual LED indicator in AmiBroker status bar\
-Supports Mixed EOD Timeframe

<figure><img src="../../.gitbook/assets/Amibroker Charts.png" alt=""><figcaption></figcaption></figure>

### Installation Steps

#### 1. Download the Plugin

Download the [OpenAlgo Amibroker Plugin](https://github.com/marketcalls/OpenAlgoPlugin/releases) from the official release page:

<figure><img src="../../.gitbook/assets/image (134).png" alt=""><figcaption></figcaption></figure>

#### 2. Extract the Plugin

Extract the downloaded `OpenAlgo.Plugin.zip` file. You'll find two folders:

* `32bit` - For 32-bit AmiBroker installations
* `64bit` - For 64-bit AmiBroker installations

<figure><img src="../../.gitbook/assets/image (133).png" alt=""><figcaption></figcaption></figure>

#### 3. Close AmiBroker

**IMPORTANT:** If AmiBroker is currently running, close it completely before proceeding. This ensures the plugin loads correctly when AmiBroker restarts.

#### 4. Copy the Plugin File

1. Navigate to the appropriate folder based on your AmiBroker version:
   * For 64-bit AmiBroker: Open the `64bit` folder
   * For 32-bit AmiBroker: Open the `32bit` folder
2. Copy the `OpenAlgo.dll` file
3.  Paste it into your AmiBroker Plugins folder, typically located at:

    ```
    C:\Program Files\AmiBroker\Plugins\
    ```

    or

    ```
    C:\Program Files (x86)\AmiBroker\Plugins\
    ```

<figure><img src="../../.gitbook/assets/image (135).png" alt=""><figcaption></figcaption></figure>

#### 5. Restart AmiBroker

Launch AmiBroker. The OpenAlgo plugin should now be loaded automatically.

### Configuration

#### 1. Create/Open AmiBroker Database

1. In AmiBroker, create a new database or open an existing one
2. When creating a database, select **OpenAlgo Plugin** as the data source
3. Set the Number of Bars as 75000 for sufficient amount of backfill data
4. Set the Base Time Interval as 1-minute

<figure><img src="../../.gitbook/assets/image (136).png" alt=""><figcaption></figcaption></figure>

#### 2. Configure Intraday Settings

1. Click on **Intraday** **Settings button**
2. Select the option: **"Allow mixed EOD/intraday data"**
3. Click **OK** to save

<figure><img src="../../.gitbook/assets/image (137).png" alt=""><figcaption></figcaption></figure>

#### 3. Configure OpenAlgo Plugin Settings

1. Click on  **Configure Button**
2. Configure the **Server Settings**:
   * **Server**: Enter your OpenAlgo server address (e.g., `127.0.0.1`)
   * **Port**: Enter the port number (default: `5000`)
   * **API Key**: Enter your OpenAlgo API key
   * **Refresh Interval**: Set the data refresh interval in seconds (default: `5`)
   * **Time Shift (hours)**: Set time shift if needed (default: `0`)
3. Click **Test Connection** to verify server connectivity
4. Configure the **WebSocket Settings**:
   * **WebSocket URL**: Enter your WebSocket URL (e.g., `ws://127.0.0.1:8765`)
   * Click **Test WebSocket** to verify the connection
   * Monitor the **WebSocket Status** field
5. Verify the **Connection Status** shows as "Ready"
6. Click **OK** to save all settings

<figure><img src="../../.gitbook/assets/image (138).png" alt=""><figcaption></figcaption></figure>

#### 4. Add Symbols

1. In AmiBroker, go to **Symbol** → **New to add anew symbol**
2. Click **New** to add a new symbol
3. Enter the symbol ticker (e.g., `SBIN-NSE`, `RELIANCE-BSE`, `NIFTY-NIFTY_INDEX`)
4. symbol format is `openalgo symbol-openalgo exchange`
5. The plugin will automatically fetch live data for the symbol

### Verification

1. Ensure OpenAlgo server is running in the background
2. Add a symbol in AmiBroker
3. Open a chart for the symbol
4. You should see live data streaming into the chart
5. Check the AmiBroker log window for any connection or data errors

### Troubleshooting

#### Plugin Not Loading

* Verify you copied the correct DLL version (32-bit or 64-bit) matching your AmiBroker installation
* Ensure AmiBroker was restarted after copying the plugin
* Check Windows Event Viewer for any DLL loading errors

#### Connection Issues

* Verify OpenAlgo server is running
* Check the Server address and Port are correct
* Ensure your firewall is not blocking the connection
* Test the API key directly with OpenAlgo API

#### WebSocket Issues

* Verify the WebSocket URL is correct
* Check if the WebSocket service is running on the OpenAlgo server
* Ensure port 8765 (or your configured port) is not blocked

#### No Data Appearing

* Verify the symbol format matches what OpenAlgo expects
* Check the AmiBroker log window for error messages
* Ensure the Refresh Interval is appropriate (not too high)
* Verify your OpenAlgo subscription includes the requested symbols

### Support

For issues and support, please visit:

* GitHub Issues: https://github.com/marketcalls/OpenAlgoPlugin/issues
* OpenAlgo Documentation: https://docs.openalgo.in

### Notes

* The plugin requires an active internet connection and running OpenAlgo server connected with your broker
* Ensure your OpenAlgo server has valid broker credentials configured
* Data availability depends on your broker's data feed and OpenAlgo subscription
* WebSocket connection provides real-time tick data updates
