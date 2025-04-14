# AmiQuotes

**This tool helps you to fetch live data ( 1minute / Daily) data directly from the broker and seamlessly update Amibroker using the Amiquotes tool every minute.**



### Prerequisites

OpenAlgo  and OpenAlgo API Key

Amibroker 6.0 or higher

Amiquotes 4.10 or higher (mostly preinstalled with Amibroker) if not download from [Amibroker Download section](https://amibroker.com/download.html)

OpenAlgo supported Brokers providing Historical Data / Intraday Data API

Note :  As of now kotak not supporting intraday/historical data API



### Step 1 : Download OpenAlgo - Amiquotes (Plugin)



{% file src="../../.gitbook/assets/OpenAlgo.ads" %}

Download the file and save under **Amibroker -> Amiquotes -> DataSource Folder**

<figure><img src="../../.gitbook/assets/image (64).png" alt=""><figcaption></figcaption></figure>



### Step 2 : Create the Amibroker Database

Open Amibroker Goto File Menu -> New -> Database

<figure><img src="../../.gitbook/assets/image (65).png" alt=""><figcaption></figcaption></figure>

Create a New Database

Enter the Database Name, Base Time Interval and Click Create

<figure><img src="../../.gitbook/assets/image (66).png" alt=""><figcaption></figcaption></figure>

Enter the Database Source as (local database) and Number of Bars as 75000

<figure><img src="../../.gitbook/assets/image (67).png" alt=""><figcaption></figcaption></figure>

Now click on Intraday Settings and Enable Allow Mixed EOD/Interval data and press ok

<figure><img src="../../.gitbook/assets/image (68).png" alt=""><figcaption></figcaption></figure>



### Step 3 : Add OpenAlgo Ticker Symbols

Add Ticker Symbols from the Symbols menu -> New

<figure><img src="../../.gitbook/assets/image (69).png" alt=""><figcaption></figcaption></figure>

You can Also add the ticker symbol with comma separated

<figure><img src="../../.gitbook/assets/image (70).png" alt=""><figcaption></figcaption></figure>

If you want to import bulk symbols consider using Watchlist import method (Supports .txt or .csv or .tls format)

<figure><img src="../../.gitbook/assets/image (71).png" alt=""><figcaption></figcaption></figure>



### Step 4 : Amiquote Realtime Update

Open Amiquote from Tools Menu -> Auto-update quotes (AmiQuote)

<figure><img src="../../.gitbook/assets/image (72).png" alt=""><figcaption></figcaption></figure>

From the dropdown of Amiquote Select OpenAlgo

<figure><img src="../../.gitbook/assets/image (73).png" alt=""><figcaption></figcaption></figure>

Alternatively, you can also import the openalgo template using the import method. Goto **Data Source Menu -> Import**

<figure><img src="../../.gitbook/assets/image (76).png" alt=""><figcaption></figcaption></figure>



configure the OpenAlgo API key

Get the OpenAlgo API Key by visiting **openalgo -> Dashboard -> Click on Profile Icon -> APIKey**

Create one if you installed openalgo for the first time. Copy the API key and paste in the User-definable data source -> API Key Section as shown below and press ok

<figure><img src="../../.gitbook/assets/image (78).png" alt=""><figcaption></figcaption></figure>

Now Select the desired interval and set Run every 1min  and press the play button

<figure><img src="../../.gitbook/assets/image (79).png" alt=""><figcaption></figcaption></figure>

Now the Data Download from the Broker via OpenAlgo Starts

<figure><img src="../../.gitbook/assets/image (80).png" alt=""><figcaption></figcaption></figure>

Now you can see that openalgo is Auto updating the Amibroker Database every 1 minute

<figure><img src="../../.gitbook/assets/image (81).png" alt=""><figcaption></figcaption></figure>

Make sure that Charts are loaded properly.

<figure><img src="../../.gitbook/assets/Amibroker Chart.png" alt=""><figcaption></figcaption></figure>

Start using your brokers data for your analysis purpose.\


For more details and troubleshooting refer the [Ticker API](../../api-documentation/v1/data-api/ticker.md)

Make sure to login to openalgo every day to fetch the live intraday/EOD quotes
