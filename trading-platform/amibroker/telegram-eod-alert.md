---
description: Works with Any Long-Only AFL Strategy
---

# Telegram EOD Alert

#### Designed Exclusively for Daily Timeframe

This document describes how to automatically run a daily AFL exploration at 6:00 PM IST and send Telegram alerts using OpenAlgo.

```
//====================================================================
// UNIVERSAL TELEGRAM EXPLORATION MODULE (LONG ONLY)
// Plug-and-Play for any AFL strategy that defines Buy & Sell signals
//====================================================================

_SECTION_BEGIN("Telegram Alerts (Long Only)");

// ---------- OpenAlgo / Telegram parameters ----------
tgApiKey    = ParamStr("OpenAlgo API Key", "your_api_key_here");
tgUserName  = ParamStr("OpenAlgo Username", "your_openalgo_username");
tgHost      = ParamStr("Host", "http://127.0.0.1:5000");
tgVer       = ParamStr("API Version", "v1");
tgPriority  = Param("Telegram Priority", 5, 1, 10, 1);

// Build endpoint URL
tgUrl = tgHost + "/api/" + tgVer + "/telegram/notify";


// ---------- Telegram sender function ----------
function SendTelegramAlert(tgMessage)
{
    global tgApiKey, tgUserName, tgPriority, tgUrl;

    pstr = StrFormat("%g", tgPriority);

    postBody =
          "{"
        + "\"apikey\":\""   + tgApiKey    + "\","
        + "\"username\":\"" + tgUserName  + "\","
        + "\"message\":\""  + tgMessage   + "\","
        + "\"priority\":"    + pstr
        + "}";

    headers =
          "Content-Type: application/json\r\n"
        + "Cache-Control: no-cache\r\n"
        + "Pragma: no-cache\r\n";

    InternetSetHeaders(headers);
    ih = InternetPostRequest(tgUrl, postBody);

    if(ih)
    {
        resp = "";
        line = "";

        while((line = InternetReadString(ih)) != "")
            resp += line;

        InternetClose(ih);
        _TRACE("Telegram Response: " + resp);
    }
    else
    {
        _TRACE("Telegram Error: InternetPostRequest failed");
    }
}



//====================================================================
// EXPLORATION BLOCK (Drop-in for any Long-Only strategy)
//====================================================================

// Show all BUY / SELL in exploration
Filter = Buy OR Sell;

// Background coloring
bgColor   = IIf(Buy, colorGreen, IIf(Sell, colorRed, colorBlack));
textColor = colorWhite;

// Exploration columns
AddTextColumn(Name(), "Symbol", 1, textColor, bgColor);
AddColumn(Close, "Close", 1.2, textColor, bgColor);
AddColumn(ROC(Close,1), "% Chg", 1.2, textColor, bgColor);
AddColumn(IIf(Buy, 1, 0), "Buy", 1, textColor, bgColor);
AddColumn(IIf(Sell,1, 0), "Sell",1, textColor, bgColor);

SetSortColumns(-2);  // Sort by % change



//====================================================================
// TELEGRAM ALERT TRIGGER (Only when running Exploration)
//====================================================================

if(Status("action") == actionExplore)
{
    lastBuy  = LastValue(Buy);
    lastSell = LastValue(Sell);

    if(lastBuy OR lastSell)
    {
        sname = Name();

        // capture latest bar prices for signals
        bp = LastValue( ValueWhen(Buy,  Close) );
        sp = LastValue( ValueWhen(Sell, Close) );

        // build message
        if(lastBuy AND NOT lastSell)
        {
            txt =
                "Symbol : " + sname + "\\n"
              + "Price : "  + StrFormat("%.2f", bp) + "\\n"
              + "Signal Type : BUY Signal";
        }
        else
        {
            txt =
                "Symbol : " + sname + "\\n"
              + "Price : "  + StrFormat("%.2f", sp) + "\\n"
              + "Signal Type : SELL Signal";
        }

        // send alert
        SendTelegramAlert(txt);

        _TRACE("Telegram Sent -> " + txt);
    }
}

_SECTION_END();

```

\
\
This module is intended for **Daily timeframe only**. Lower timeframes are **not supported** because they require special alert suppression logic.

***

## 1. Telegram Bot Setup (One-Time Process)

This step must be completed before AmiBroker can send alerts to your Telegram.

### 1.1 Create a Telegram Bot

1. Open the Telegram app.
2. Search for “BotFather”.
3. Send the command:

```
/newbot
```

4. Choose a bot name and username.
5. BotFather will respond with:

```
HTTP API Token: 1234567890:ABCDEF...
```

Save this token.

### 1.2 Configure the Bot Token in OpenAlgo

1. Open the OpenAlgo desktop application.
2. Go to “Telegram Settings”.
3. Paste the Bot Token received from BotFather.
4. Click “Start Bot”.

Your bot is now active.

### 1.3 Link Your Telegram Account with OpenAlgo

In Telegram, open your bot and send:

```
/link <your-openalgo-api-key> http://127.0.0.1:5000
```

Example:

```
/link ABCD1234XYZ http://127.0.0.1:5000
```

If successful, the bot will respond confirming the link.\
Your Telegram account is now authenticated with OpenAlgo and ready to receive alerts.

***

## 2. Add the Telegram Exploration Module to Your AFL Strategy

Your strategy must contain valid daily timeframe Buy and Sell signals:

```afl
Buy  = ...;     // Daily timeframe signal
Sell = ...;     // Daily timeframe signal
```

Below your strategy code, insert the **Long-Only Daily Telegram Exploration Module**.

This module automatically:

* Detects Buy/Sell signals on the _latest daily bar_
* Sends formatted alerts via OpenAlgo when Exploration is executed

The alert format is:

```
Symbol : RELIANCE
Price : 1542.00
Signal Type : BUY Signal
```

This module is intended for **Daily timeframe only**.

***

## 3. Create an APX (Analysis Settings File)

1. Open AmiBroker’s Analysis window.
2. Load your AFL containing Buy/Sell signals and the Telegram module.
3. Set the following:

* Apply To: Filter or All Symbols
* Range: 1 recent day(s)
* From/To: Today
* Timeframe: Daily

4. Click “Explore” to verify signals display as expected.
5. Save the analysis settings:

```
File → Save As…
```

Save as:

```
MyStrategy_Explore.apx
```

The APX stores:

* Formulas
* Symbol list
* Date range
* Exploration settings

***

## 4. Create a Batch File (.abb)

1. Go to:

```
File → New → Batch
```

2. Click “Insert”.
3. Select Action: **Explore**.
4. Browse and choose your `.apx` file.
5. Save the batch as:

```
MyStrategy_Batch.abb
```

This batch file will run exploration automatically.

***

## 5. Schedule Automatic Daily Execution at 6 PM IST

1. Open:

```
Tools → Scheduler
```

2. Click “Add Task”.
3. Select your batch file.
4. Configure:

* At specific date/time: 18:00:00
* Repeat: Daily (Monday–Friday)

5. Click “OK”.

At the scheduled time, AmiBroker will automatically execute:

* The batch
* The exploration
* The Telegram alert module
* The alerts will be delivered to your linked Telegram account via OpenAlgo

***

## 6. Daily Execution Workflow

Every day at 6 PM IST:

1. AmiBroker loads the APX
2. Runs the daily exploration
3. Detects Buy/Sell signals from today’s daily bar
4. Sends Telegram alerts via OpenAlgo
5. No manual intervention is required

This works only on **Daily timeframe** strategies.

***

## Notes and Restrictions

1. This module is **exclusively designed for Daily timeframe**.
2. Intraday or lower timeframes are **not supported**, because they require:
   * Bar-level alert deduplication
   * StaticVar suppression
   * Session segmentation
3. AmiBroker must remain running for Scheduler tasks to execute.
4. OpenAlgo must remain running for Telegram API to respond.
5. Internet connection must remain active.

***

## Summary

| Step | Description                                     | Output                  |
| ---- | ----------------------------------------------- | ----------------------- |
| 1    | Create Telegram bot + authenticate via OpenAlgo | Bot ready               |
| 2    | Add Telegram module to strategy AFL             | AFL with alerts         |
| 3    | Save analysis settings as APX                   | MyStrategy\_Explore.apx |
| 4    | Map APX in Batch file                           | MyStrategy\_Batch.abb   |
| 5    | Schedule daily at 6 PM IST                      | Automatic daily alerts  |

When combined, this provides a reliable fully automated **Daily timeframe alerting system** using AmiBroker + OpenAlgo + Telegram.

