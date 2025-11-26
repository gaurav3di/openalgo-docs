# Limit Order Execution

### OpenAlgo Limit Order Execution Module

The OpenAlgo Limit Order Execution Module is an AmiBroker AFL-based trading tool designed to automate limit order placement with intelligent price improvement and retry logic. It connects to the OpenAlgo API to fetch real-time bid/ask quotes and execute limit orders at calculated prices based on five configurable modes: Mid-Price, Best Bid, Best Ask, Bid Offset, and Ask Offset. The module features a visual GFX dashboard that displays live market data including bid, ask, LTP, mid-price, and spread, along with complete order status tracking showing the current state, order ID, execution price, and retry count.

<figure><img src="../../.gitbook/assets/Amibroker Charts.png" alt=""><figcaption></figcaption></figure>

The execution workflow follows a state machine architecture that places a limit order at the calculated price, monitors order status, and automatically modifies the order with fresh quotes if it remains unfilled. This modify-and-retry loop continues up to a configurable number of attempts (default: 3), after which the pending order is automatically canceled. The module supports configurable tick sizes for proper price rounding across different instruments (stocks, futures, commodities), making it suitable for exchanges like NSE, NFO, MCX, and others. Users can trigger BUY or SELL orders via parameter buttons, manually cancel active orders, or reset the module state as needed.

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

### Amibroker Execution Module

```c
_SECTION_BEGIN("OpenAlgo Limit Order Execution Module");

RequestTimedRefresh(5, False);

/* ******************************************************************** */
/* USER PARAMETERS */
/* ******************************************************************** */
apiKey = ParamStr("API Key", "your_api_key_here");
symbolNm = ParamStr("Symbol", "CRUDEOIL18DEC25FUT");
exchangeNm = ParamList("Exchange", "MCX|NSE|NFO|CDS|BSE|BFO|BCD");
strategyNm = ParamStr("Strategy Name", "LimitExec");
quantity = Param("Quantity", 1, 1, 10000, 1);
prdType = ParamList("Product", "MIS|CNC|NRML");

/* Execution Settings */
priceMode = ParamList("Price Mode", "MidPrice|BestBid|BestAsk|BidOffset|AskOffset");
tickOffset = Param("Tick Offset", 1, 0, 10, 1);
tickSizeStr = ParamList("Tick Size", "1|0.01|0.05|0.1|0.2|0.25|0.5|5|10|25");
tickSize = StrToNum(tickSizeStr);
maxRetries = Param("Max Retries", 3, 1, 10, 1);
retryDelaySec = Param("Retry Delay (Sec)", 1, 1, 10, 1);

/* Trigger Buttons */
triggerBuy = ParamTrigger("Place BUY Limit", "BUY NOW");
triggerSell = ParamTrigger("Place SELL Limit", "SELL NOW");
triggerCancel = ParamTrigger("Cancel Order", "CANCEL");
triggerReset = ParamTrigger("Reset Module", "RESET");

/* GFX Settings */
cellHeight = Param("Cell Height", 30, 20, 50, 1);
cellWidth = Param("Cell Width", 130, 80, 200, 1);
startX = Param("Start X", 50, 0, 1200, 1);
startY = Param("Start Y", 80, 0, 1200, 1);

/* Colors */
headerBgColor = ParamColor("Header BG", colorDarkBlue);
headerTextColor = ParamColor("Header Text", colorWhite);
bidBgColor = ParamColor("Bid BG", colorDarkGreen);
askBgColor = ParamColor("Ask BG", colorDarkRed);
infoBgColor = ParamColor("Info BG", colorGrey40);
infoTextColor = ParamColor("Info Text", colorWhite);

/* State Constants */
STATE_IDLE = 0;
STATE_PLACING = 1;
STATE_MONITORING = 2;
STATE_MODIFYING = 3;
STATE_CANCELING = 4;
STATE_FINAL_CHECK = 5;
STATE_COMPLETE = 6;
STATE_CANCELED = 7;
STATE_REJECTED = 8;

/* ******************************************************************** */
/* CHECK API KEY VALIDITY */
/* ******************************************************************** */
apiKeyValid = True;
if(apiKey == "your_api_key_here" OR StrLen(apiKey) < 10)
{
    apiKeyValid = False;
}

/* ******************************************************************** */
/* ROUND PRICE TO NEAREST TICK SIZE */
/* ******************************************************************** */
function RoundToTick(price, tick)
{
    roundedPrice = round(price / tick) * tick;
    return roundedPrice;
}

/* ******************************************************************** */
/* JSON EXTRACTION FUNCTION */
/* ******************************************************************** */
function ExtractJsonValue(jsonStr, keyName, isNested)
{
    result = "";
    jsonData = jsonStr;

    if(isNested)
    {
        dataStart = StrFind(jsonStr, "\"data\":{");
        if(dataStart > 0) dataStart = dataStart - 1;
        
        if(dataStart >= 0)
        {
            valueStart = dataStart + 7;
            valueEnd = valueStart;
            braceCount = 1;
            strLenJson = StrLen(jsonStr);

            while(valueEnd < strLenJson)
            {
                currChar = StrMid(jsonStr, valueEnd, 1);
                if(currChar == "{") braceCount++;
                if(currChar == "}") braceCount--;
                if(braceCount == 0) break;
                valueEnd++;
            }
            
            jsonData = StrMid(jsonStr, valueStart, valueEnd - valueStart);
        }
    }
    
    keyPos = StrFind(jsonData, "\"" + keyName + "\":");
    if(keyPos > 0) keyPos = keyPos - 1;
    
    if(keyPos > -1)
    {
        valueStart = keyPos + StrLen(keyName) + 3;
        while(StrMid(jsonData, valueStart, 1) == " ") valueStart++;
        
        firstValChar = StrMid(jsonData, valueStart, 1);
        isQuoted = (firstValChar == "\"");
        
        strLenData = StrLen(jsonData);
        valueEnd = valueStart;
        
        if(isQuoted)
        {
            valueStart++;
            valueEnd = valueStart;
            while(valueEnd < strLenData)
            {
                currChar = StrMid(jsonData, valueEnd, 1);
                if(currChar == "\"") break;
                valueEnd++;
            }
            result = StrMid(jsonData, valueStart, valueEnd - valueStart);
        }
        else
        {
            while(valueEnd < strLenData)
            {
                currChar = StrMid(jsonData, valueEnd, 1);
                if(currChar == "," OR currChar == "}") break;
                valueEnd++;
            }
            result = StrMid(jsonData, valueStart, valueEnd - valueStart);
        }
    }
    
    return result;
}

/* ******************************************************************** */
/* GFX TABLE CELL FUNCTION */
/* ******************************************************************** */
function TableCell(text, col, row, bgColor, textColor)
{
    x1 = startX + (col - 1) * cellWidth;
    y1 = startY + (row - 1) * cellHeight;
    x2 = x1 + cellWidth;
    y2 = y1 + cellHeight;
    textX = x1 + (cellWidth / 2);
    textY = y1 + (cellHeight / 2) - 6;

    GfxSelectPen(colorBlack, 1);
    GfxSelectSolidBrush(bgColor);
    GfxRectangle(x1, y1, x2, y2);

    GfxSetBkColor(bgColor);
    GfxSetTextAlign(6);
    GfxSetTextColor(textColor);
    GfxTextOut(text, textX, textY);
    
    return 1;
}

/* ******************************************************************** */
/* WIDE TABLE CELL FUNCTION */
/* ******************************************************************** */
function TableCellWide(text, col, row, colSpan, bgColor, textColor)
{
    x1 = startX + (col - 1) * cellWidth;
    y1 = startY + (row - 1) * cellHeight;
    x2 = x1 + (cellWidth * colSpan);
    y2 = y1 + cellHeight;
    textX = x1 + ((cellWidth * colSpan) / 2);
    textY = y1 + (cellHeight / 2) - 6;

    GfxSelectPen(colorBlack, 1);
    GfxSelectSolidBrush(bgColor);
    GfxRectangle(x1, y1, x2, y2);

    GfxSetBkColor(bgColor);
    GfxSetTextAlign(6);
    GfxSetTextColor(textColor);
    GfxTextOut(text, textX, textY);
    
    return 1;
}

/* ******************************************************************** */
/* API: FETCH QUOTES */
/* ******************************************************************** */
function FetchQuotes(apiKeyLocal, sym, exch)
{
    returnValue = 0;
    urlMain = "http://127.0.0.1:5000/api/v1/quotes";
    
    postBody = "{";
    postBody += "\"apikey\":\"" + apiKeyLocal + "\",";
    postBody += "\"symbol\":\"" + sym + "\",";
    postBody += "\"exchange\":\"" + exch + "\"";
    postBody += "}";
    
    InternetSetHeaders("Content-Type: application/json\r\n");
    ih = InternetPostRequest(urlMain, postBody);
    
    if(ih)
    {
        rsp = "";
        respLine = InternetReadString(ih);
        while(respLine != "")
        {
            rsp += respLine;
            respLine = InternetReadString(ih);
        }
        InternetClose(ih);
        
        _TRACEF("Quotes Response: %s", rsp);
        
        statusVal = ExtractJsonValue(rsp, "status", False);
        if(StrFind(statusVal, "success") > 0)
        {
            bidVal = StrToNum(ExtractJsonValue(rsp, "bid", True));
            askVal = StrToNum(ExtractJsonValue(rsp, "ask", True));
            ltpVal = StrToNum(ExtractJsonValue(rsp, "ltp", True));
            
            StaticVarSet("LimitExec_Bid", bidVal);
            StaticVarSet("LimitExec_Ask", askVal);
            StaticVarSet("LimitExec_LTP", ltpVal);
            StaticVarSet("LimitExec_QuoteTime", Now(4));
            
            returnValue = 1;
        }
    }
    else
    {
        _TRACE("FetchQuotes Failed");
    }
    
    return returnValue;
}

/* ******************************************************************** */
/* API: PLACE LIMIT ORDER */
/* ******************************************************************** */
function PlaceLimitOrder(apiKeyLocal, strat, sym, exch, action, prdParam, qty, limitPrice)
{
    returnValue = 0;
    urlMain = "http://127.0.0.1:5000/api/v1/placeorder";
    
    postBody = "{";
    postBody += "\"apikey\":\"" + apiKeyLocal + "\",";
    postBody += "\"strategy\":\"" + strat + "\",";
    postBody += "\"symbol\":\"" + sym + "\",";
    postBody += "\"exchange\":\"" + exch + "\",";
    postBody += "\"action\":\"" + action + "\",";
    postBody += "\"product\":\"" + prdParam + "\",";
    postBody += "\"pricetype\":\"LIMIT\",";
    postBody += "\"quantity\":\"" + NumToStr(qty, 1.0, False) + "\",";
    postBody += "\"price\":\"" + NumToStr(limitPrice, 1.2, False) + "\",";
    postBody += "\"trigger_price\":\"0\",";
    postBody += "\"disclosed_quantity\":\"0\"";
    postBody += "}";
    
    _TRACEF("PlaceOrder Request: %s", postBody);
    
    InternetSetHeaders("Content-Type: application/json\r\n");
    ih = InternetPostRequest(urlMain, postBody);
    
    if(ih)
    {
        rsp = "";
        respLine = InternetReadString(ih);
        while(respLine != "")
        {
            rsp += respLine;
            respLine = InternetReadString(ih);
        }
        InternetClose(ih);
        
        _TRACEF("PlaceOrder Response: %s", rsp);
        
        statusVal = ExtractJsonValue(rsp, "status", False);
        if(StrFind(statusVal, "success") > 0)
        {
            orderID = ExtractJsonValue(rsp, "orderid", False);
            StaticVarSetText("LimitExec_OrderID", orderID);
            returnValue = 1;
        }
    }
    else
    {
        _TRACE("PlaceLimitOrder Failed");
    }
    
    return returnValue;
}

/* ******************************************************************** */
/* API: CHECK ORDER STATUS */
/* ******************************************************************** */
function CheckOrderStatus(apiKeyLocal, strat, orderID)
{
    returnValue = 0;
    urlMain = "http://127.0.0.1:5000/api/v1/orderstatus";
    
    postBody = "{";
    postBody += "\"apikey\":\"" + apiKeyLocal + "\",";
    postBody += "\"strategy\":\"" + strat + "\",";
    postBody += "\"orderid\":\"" + orderID + "\"";
    postBody += "}";
    
    InternetSetHeaders("Content-Type: application/json\r\n");
    ih = InternetPostRequest(urlMain, postBody);
    
    if(ih)
    {
        rsp = "";
        respLine = InternetReadString(ih);
        while(respLine != "")
        {
            rsp += respLine;
            respLine = InternetReadString(ih);
        }
        InternetClose(ih);
        
        _TRACEF("OrderStatus Response: %s", rsp);
        
        statusVal = ExtractJsonValue(rsp, "status", False);
        if(StrFind(statusVal, "success") > 0)
        {
            orderStatus = ExtractJsonValue(rsp, "order_status", True);
            avgPrice = StrToNum(ExtractJsonValue(rsp, "average_price", True));
            
            StaticVarSetText("LimitExec_OrderStatus", orderStatus);
            StaticVarSet("LimitExec_FilledPrice", avgPrice);
            
            returnValue = 1;
        }
    }
    else
    {
        _TRACE("CheckOrderStatus Failed");
    }
    
    return returnValue;
}

/* ******************************************************************** */
/* API: MODIFY ORDER */
/* ******************************************************************** */
function ModifyLimitOrder(apiKeyLocal, strat, sym, exch, action, orderID, prdParam, qty, newPrice)
{
    returnValue = 0;
    urlMain = "http://127.0.0.1:5000/api/v1/modifyorder";
    
    postBody = "{";
    postBody += "\"apikey\":\"" + apiKeyLocal + "\",";
    postBody += "\"strategy\":\"" + strat + "\",";
    postBody += "\"symbol\":\"" + sym + "\",";
    postBody += "\"exchange\":\"" + exch + "\",";
    postBody += "\"action\":\"" + action + "\",";
    postBody += "\"orderid\":\"" + orderID + "\",";
    postBody += "\"product\":\"" + prdParam + "\",";
    postBody += "\"pricetype\":\"LIMIT\",";
    postBody += "\"quantity\":\"" + NumToStr(qty, 1.0, False) + "\",";
    postBody += "\"price\":\"" + NumToStr(newPrice, 1.2, False) + "\",";
    postBody += "\"trigger_price\":\"0\",";
    postBody += "\"disclosed_quantity\":\"0\"";
    postBody += "}";
    
    _TRACEF("ModifyOrder Request: %s", postBody);
    
    InternetSetHeaders("Content-Type: application/json\r\n");
    ih = InternetPostRequest(urlMain, postBody);
    
    if(ih)
    {
        rsp = "";
        respLine = InternetReadString(ih);
        while(respLine != "")
        {
            rsp += respLine;
            respLine = InternetReadString(ih);
        }
        InternetClose(ih);
        
        _TRACEF("ModifyOrder Response: %s", rsp);
        
        statusVal = ExtractJsonValue(rsp, "status", False);
        if(StrFind(statusVal, "success") > 0)
        {
            returnValue = 1;
        }
    }
    else
    {
        _TRACE("ModifyLimitOrder Failed");
    }
    
    return returnValue;
}

/* ******************************************************************** */
/* API: CANCEL ORDER */
/* ******************************************************************** */
function CancelLimitOrder(apiKeyLocal, strat, orderID)
{
    returnValue = 0;
    urlMain = "http://127.0.0.1:5000/api/v1/cancelorder";
    
    postBody = "{";
    postBody += "\"apikey\":\"" + apiKeyLocal + "\",";
    postBody += "\"strategy\":\"" + strat + "\",";
    postBody += "\"orderid\":\"" + orderID + "\"";
    postBody += "}";
    
    _TRACEF("CancelOrder Request: %s", postBody);
    
    InternetSetHeaders("Content-Type: application/json\r\n");
    ih = InternetPostRequest(urlMain, postBody);
    
    if(ih)
    {
        rsp = "";
        respLine = InternetReadString(ih);
        while(respLine != "")
        {
            rsp += respLine;
            respLine = InternetReadString(ih);
        }
        InternetClose(ih);
        
        _TRACEF("CancelOrder Response: %s", rsp);
        
        statusVal = ExtractJsonValue(rsp, "status", False);
        if(StrFind(statusVal, "success") > 0)
        {
            returnValue = 1;
        }
    }
    else
    {
        _TRACE("CancelLimitOrder Failed");
    }
    
    return returnValue;
}

/* ******************************************************************** */
/* CALCULATE LIMIT PRICE BASED ON MODE */
/* ******************************************************************** */
function CalculateLimitPrice(action, mode, bid, ask, offsetTicks, tick)
{
    midPrice = (bid + ask) / 2;
    calcPrice = midPrice;
    
    if(mode == "MidPrice")
    {
        calcPrice = midPrice;
    }
    
    if(mode == "BestBid")
    {
        calcPrice = bid;
    }
    
    if(mode == "BestAsk")
    {
        calcPrice = ask;
    }
    
    if(mode == "BidOffset")
    {
        calcPrice = bid + (offsetTicks * tick);
    }
    
    if(mode == "AskOffset")
    {
        calcPrice = ask - (offsetTicks * tick);
    }
    
    calcPrice = round(calcPrice / tick) * tick;
    
    return calcPrice;
}

/* ******************************************************************** */
/* INITIALIZE STATIC VARIABLES */
/* ******************************************************************** */
if(IsNull(StaticVarGet("LimitExec_State")))
{
    StaticVarSet("LimitExec_State", STATE_IDLE);
    StaticVarSetText("LimitExec_OrderID", "");
    StaticVarSetText("LimitExec_Action", "");
    StaticVarSetText("LimitExec_OrderStatus", "");
    StaticVarSetText("LimitExec_Message", "Ready");
    StaticVarSet("LimitExec_RetryCount", 0);
    StaticVarSet("LimitExec_LastActionTime", 0);
    StaticVarSet("LimitExec_OrderPrice", 0);
    StaticVarSet("LimitExec_FilledPrice", 0);
    StaticVarSet("LimitExec_Bid", 0);
    StaticVarSet("LimitExec_Ask", 0);
    StaticVarSet("LimitExec_LTP", 0);
    StaticVarSet("LimitExec_MidPrice", 0);
}

/* ******************************************************************** */
/* HANDLE RESET TRIGGER */
/* ******************************************************************** */
if(triggerReset)
{
    StaticVarSet("LimitExec_State", STATE_IDLE);
    StaticVarSetText("LimitExec_OrderID", "");
    StaticVarSetText("LimitExec_Action", "");
    StaticVarSetText("LimitExec_OrderStatus", "");
    StaticVarSetText("LimitExec_Message", "Module Reset");
    StaticVarSet("LimitExec_RetryCount", 0);
    StaticVarSet("LimitExec_LastActionTime", 0);
    StaticVarSet("LimitExec_OrderPrice", 0);
    StaticVarSet("LimitExec_FilledPrice", 0);
    _TRACE("=== MODULE RESET ===");
}

/* ******************************************************************** */
/* RETRIEVE CURRENT STATE */
/* ******************************************************************** */
currentState = StaticVarGet("LimitExec_State");
currentTime = Now(4);
lastActionTime = StaticVarGet("LimitExec_LastActionTime");
retryCount = StaticVarGet("LimitExec_RetryCount");
orderID = StaticVarGetText("LimitExec_OrderID");
actionType = StaticVarGetText("LimitExec_Action");

/* ******************************************************************** */
/* MAIN LOGIC - ONLY EXECUTE IF API KEY IS VALID */
/* ******************************************************************** */
if(apiKeyValid)
{
    /* ******************************************************************** */
    /* FETCH QUOTES PERIODICALLY */
    /* ******************************************************************** */
    quoteRefreshTime = StaticVarGet("LimitExec_QuoteTime");
    if(IsNull(quoteRefreshTime)) quoteRefreshTime = 0;

    if(currentTime - quoteRefreshTime >= 2)
    {
        FetchQuotes(apiKey, symbolNm, exchangeNm);
    }

    /* Calculate Mid Price with Tick Rounding */
    bidPrice = StaticVarGet("LimitExec_Bid");
    askPrice = StaticVarGet("LimitExec_Ask");
    ltpPrice = StaticVarGet("LimitExec_LTP");
    midPriceRaw = (bidPrice + askPrice) / 2;
    midPrice = RoundToTick(midPriceRaw, tickSize);
    StaticVarSet("LimitExec_MidPrice", midPrice);

    /* ******************************************************************** */
    /* HANDLE BUY/SELL TRIGGERS */
    /* ******************************************************************** */
    if(triggerBuy AND currentState == STATE_IDLE)
    {
        StaticVarSetText("LimitExec_Action", "BUY");
        StaticVarSet("LimitExec_State", STATE_PLACING);
        StaticVarSet("LimitExec_RetryCount", 0);
        StaticVarSetText("LimitExec_Message", "Initiating BUY order...");
        _TRACE("=== BUY TRIGGERED ===");
    }

    if(triggerSell AND currentState == STATE_IDLE)
    {
        StaticVarSetText("LimitExec_Action", "SELL");
        StaticVarSet("LimitExec_State", STATE_PLACING);
        StaticVarSet("LimitExec_RetryCount", 0);
        StaticVarSetText("LimitExec_Message", "Initiating SELL order...");
        _TRACE("=== SELL TRIGGERED ===");
    }

    /* ******************************************************************** */
    /* HANDLE MANUAL CANCEL TRIGGER */
    /* ******************************************************************** */
    if(triggerCancel AND currentState != STATE_IDLE AND StrLen(orderID) > 0)
    {
        StaticVarSet("LimitExec_State", STATE_CANCELING);
        StaticVarSetText("LimitExec_Message", "Manual cancel requested...");
        _TRACE("=== MANUAL CANCEL TRIGGERED ===");
    }

    /* ******************************************************************** */
    /* STATE MACHINE PROCESSING */
    /* ******************************************************************** */
    currentState = StaticVarGet("LimitExec_State");
    actionType = StaticVarGetText("LimitExec_Action");

    /* STATE: PLACING */
    if(currentState == STATE_PLACING)
    {
        FetchQuotes(apiKey, symbolNm, exchangeNm);
        bidPrice = StaticVarGet("LimitExec_Bid");
        askPrice = StaticVarGet("LimitExec_Ask");
        
        limitPrice = CalculateLimitPrice(actionType, priceMode, bidPrice, askPrice, tickOffset, tickSize);
        StaticVarSet("LimitExec_OrderPrice", limitPrice);
        
        successPlace = PlaceLimitOrder(apiKey, strategyNm, symbolNm, exchangeNm, actionType, prdType, quantity, limitPrice);
        
        if(successPlace)
        {
            StaticVarSet("LimitExec_State", STATE_MONITORING);
            StaticVarSet("LimitExec_LastActionTime", currentTime);
            StaticVarSetText("LimitExec_Message", "Order placed. Monitoring...");
            _TRACEF("Order Placed: %s @ %.2f", StaticVarGetText("LimitExec_OrderID"), limitPrice);
        }
        else
        {
            StaticVarSet("LimitExec_State", STATE_REJECTED);
            StaticVarSetText("LimitExec_Message", "Order placement failed!");
        }
    }

    /* STATE: MONITORING */
    if(currentState == STATE_MONITORING)
    {
        if(currentTime - lastActionTime >= retryDelaySec)
        {
            orderID = StaticVarGetText("LimitExec_OrderID");
            successStatus = CheckOrderStatus(apiKey, strategyNm, orderID);
            
            if(successStatus)
            {
                orderStatus = StaticVarGetText("LimitExec_OrderStatus");
                
                if(StrFind(orderStatus, "complete") > 0)
                {
                    StaticVarSet("LimitExec_State", STATE_COMPLETE);
                    StaticVarSetText("LimitExec_Message", "Order FILLED!");
                    _TRACE("=== ORDER COMPLETE ===");
                }
                else if(StrFind(orderStatus, "rejected") > 0 OR StrFind(orderStatus, "cancelled") > 0)
                {
                    StaticVarSet("LimitExec_State", STATE_REJECTED);
                    StaticVarSetText("LimitExec_Message", "Order REJECTED/CANCELLED");
                    _TRACE("=== ORDER REJECTED ===");
                }
                else if(StrFind(orderStatus, "open") > 0 OR StrFind(orderStatus, "pending") > 0)
                {
                    retryCount = StaticVarGet("LimitExec_RetryCount");
                    
                    if(retryCount < maxRetries)
                    {
                        StaticVarSet("LimitExec_State", STATE_MODIFYING);
                        StaticVarSetText("LimitExec_Message", StrFormat("Open. Retry %g of %g...", retryCount + 1, maxRetries));
                    }
                    else
                    {
                        StaticVarSet("LimitExec_State", STATE_CANCELING);
                        StaticVarSetText("LimitExec_Message", "Max retries. Canceling...");
                        _TRACE("=== MAX RETRIES - CANCELING ===");
                    }
                }
            }
            
            StaticVarSet("LimitExec_LastActionTime", currentTime);
        }
    }

    /* STATE: MODIFYING */
    if(currentState == STATE_MODIFYING)
    {
        FetchQuotes(apiKey, symbolNm, exchangeNm);
        bidPrice = StaticVarGet("LimitExec_Bid");
        askPrice = StaticVarGet("LimitExec_Ask");
        
        actionType = StaticVarGetText("LimitExec_Action");
        newPrice = CalculateLimitPrice(actionType, priceMode, bidPrice, askPrice, tickOffset, tickSize);
        StaticVarSet("LimitExec_OrderPrice", newPrice);
        
        orderID = StaticVarGetText("LimitExec_OrderID");
        successModify = ModifyLimitOrder(apiKey, strategyNm, symbolNm, exchangeNm, actionType, orderID, prdType, quantity, newPrice);
        
        if(successModify)
        {
            retryCount = StaticVarGet("LimitExec_RetryCount");
            StaticVarSet("LimitExec_RetryCount", retryCount + 1);
            StaticVarSet("LimitExec_State", STATE_MONITORING);
            StaticVarSet("LimitExec_LastActionTime", currentTime);
            StaticVarSetText("LimitExec_Message", StrFormat("Modified @ %.2f. Monitoring...", newPrice));
            _TRACEF("Order Modified: Retry %g @ %.2f", retryCount + 1, newPrice);
        }
        else
        {
            StaticVarSet("LimitExec_State", STATE_CANCELING);
            StaticVarSetText("LimitExec_Message", "Modify failed. Canceling...");
        }
    }

    /* STATE: CANCELING */
    if(currentState == STATE_CANCELING)
    {
        orderID = StaticVarGetText("LimitExec_OrderID");
        CancelLimitOrder(apiKey, strategyNm, orderID);
        
        StaticVarSet("LimitExec_State", STATE_FINAL_CHECK);
        StaticVarSet("LimitExec_LastActionTime", currentTime);
        StaticVarSetText("LimitExec_Message", "Cancel sent. Final check...");
        _TRACE("Cancel request sent");
    }

    /* STATE: FINAL_CHECK */
    if(currentState == STATE_FINAL_CHECK)
    {
        if(currentTime - lastActionTime >= retryDelaySec)
        {
            orderID = StaticVarGetText("LimitExec_OrderID");
            successFinal = CheckOrderStatus(apiKey, strategyNm, orderID);
            
            if(successFinal)
            {
                orderStatus = StaticVarGetText("LimitExec_OrderStatus");
                
                if(StrFind(orderStatus, "complete") > 0)
                {
                    StaticVarSet("LimitExec_State", STATE_COMPLETE);
                    StaticVarSetText("LimitExec_Message", "Filled during cancel!");
                    _TRACE("=== FILLED DURING CANCEL ===");
                }
                else
                {
                    StaticVarSet("LimitExec_State", STATE_CANCELED);
                    StaticVarSetText("LimitExec_Message", "Order CANCELED");
                    _TRACE("=== ORDER CANCELED ===");
                }
            }
            else
            {
                StaticVarSet("LimitExec_State", STATE_CANCELED);
                StaticVarSetText("LimitExec_Message", "Assumed Canceled");
            }
        }
    }
}
else
{
    StaticVarSetText("LimitExec_Message", "Please configure valid API Key");
}

/* ******************************************************************** */
/* READ FINAL STATE FOR DISPLAY */
/* ******************************************************************** */
currentState = StaticVarGet("LimitExec_State");
orderID = StaticVarGetText("LimitExec_OrderID");
actionType = StaticVarGetText("LimitExec_Action");
orderStatus = StaticVarGetText("LimitExec_OrderStatus");
message = StaticVarGetText("LimitExec_Message");
retryCount = StaticVarGet("LimitExec_RetryCount");
orderPrice = StaticVarGet("LimitExec_OrderPrice");
filledPrice = StaticVarGet("LimitExec_FilledPrice");
bidPrice = StaticVarGet("LimitExec_Bid");
askPrice = StaticVarGet("LimitExec_Ask");
ltpPrice = StaticVarGet("LimitExec_LTP");
midPrice = StaticVarGet("LimitExec_MidPrice");
spreadVal = askPrice - bidPrice;

stateText = "IDLE";
if(currentState == STATE_PLACING) stateText = "PLACING";
if(currentState == STATE_MONITORING) stateText = "MONITORING";
if(currentState == STATE_MODIFYING) stateText = "MODIFYING";
if(currentState == STATE_CANCELING) stateText = "CANCELING";
if(currentState == STATE_FINAL_CHECK) stateText = "FINAL CHECK";
if(currentState == STATE_COMPLETE) stateText = "COMPLETE";
if(currentState == STATE_CANCELED) stateText = "CANCELED";
if(currentState == STATE_REJECTED) stateText = "REJECTED";

stateBgColor = infoBgColor;
if(currentState == STATE_COMPLETE) stateBgColor = colorDarkGreen;
if(currentState == STATE_CANCELED OR currentState == STATE_REJECTED) stateBgColor = colorDarkRed;
if(currentState == STATE_MONITORING OR currentState == STATE_MODIFYING) stateBgColor = colorDarkYellow;

/* ******************************************************************** */
/* RENDER GFX DASHBOARD */
/* ******************************************************************** */
GfxSetBkMode(1);
GfxSelectFont("Arial", 14, 700);
GfxSetTextColor(colorWhite);
GfxTextOut("LIMIT ORDER EXECUTION - " + symbolNm + " (" + exchangeNm + ")", startX, startY - 25);

TableCellWide("MARKET DATA", 1, 1, 4, headerBgColor, headerTextColor);

TableCell("BID", 1, 2, headerBgColor, headerTextColor);
TableCellWide("LTP", 2, 2, 2, headerBgColor, headerTextColor);
TableCell("ASK", 4, 2, headerBgColor, headerTextColor);

TableCell(StrFormat("%.2f", bidPrice), 1, 3, bidBgColor, colorWhite);
TableCellWide(StrFormat("%.2f", ltpPrice), 2, 3, 2, infoBgColor, colorYellow);
TableCell(StrFormat("%.2f", askPrice), 4, 3, askBgColor, colorWhite);

TableCell("MID", 1, 4, headerBgColor, headerTextColor);
TableCell(StrFormat("%.2f", midPrice), 2, 4, infoBgColor, infoTextColor);
TableCell("SPREAD", 3, 4, headerBgColor, headerTextColor);
TableCell(StrFormat("%.2f", spreadVal), 4, 4, infoBgColor, infoTextColor);

TableCellWide("", 1, 5, 4, colorBlack, colorWhite);

TableCellWide("ORDER STATUS", 1, 6, 4, headerBgColor, headerTextColor);

TableCell("STATE", 1, 7, headerBgColor, headerTextColor);
TableCellWide("ORDER ID", 2, 7, 3, headerBgColor, headerTextColor);

TableCell(stateText, 1, 8, stateBgColor, colorWhite);
displayOrderID = orderID;
if(StrLen(orderID) == 0) displayOrderID = "---";
TableCellWide(displayOrderID, 2, 8, 3, infoBgColor, infoTextColor);

TableCell("ACTION", 1, 9, headerBgColor, headerTextColor);
TableCell("PRICE", 2, 9, headerBgColor, headerTextColor);
TableCell("RETRY", 3, 9, headerBgColor, headerTextColor);
TableCell("FILLED", 4, 9, headerBgColor, headerTextColor);

displayAction = actionType;
if(StrLen(actionType) == 0) displayAction = "---";
actionBgColor = infoBgColor;
if(actionType == "BUY") actionBgColor = bidBgColor;
if(actionType == "SELL") actionBgColor = askBgColor;
TableCell(displayAction, 1, 10, actionBgColor, colorWhite);
TableCell(StrFormat("%.2f", orderPrice), 2, 10, infoBgColor, infoTextColor);
TableCell(StrFormat("%g / %g", retryCount, maxRetries), 3, 10, infoBgColor, infoTextColor);
filledDisplay = "---";
if(filledPrice > 0) filledDisplay = StrFormat("%.2f", filledPrice);
TableCell(filledDisplay, 4, 10, infoBgColor, colorGreen);

TableCellWide("", 1, 11, 4, colorBlack, colorWhite);

TableCellWide(message, 1, 12, 4, colorGrey50, colorYellow);

GfxSelectFont("Arial", 9, 400);
GfxSetTextColor(colorGrey50);
GfxTextOut("Mode: " + priceMode + " | Tick: " + tickSizeStr + " | Qty: " + NumToStr(quantity, 1.0) + " | Product: " + prdType, startX, startY + (13 * cellHeight));

/* API Key Warning */
if(NOT apiKeyValid)
{
    GfxSelectFont("Arial", 12, 700);
    GfxSetTextColor(colorRed);
    GfxTextOut("WARNING: Please configure a valid API Key in Parameters", startX, startY + (14 * cellHeight) + 10);
}

_SECTION_END();


_SECTION_BEGIN("Candle with X-Axis");

SetChartOptions(0, chartShowArrows | chartShowDates);

_N(Title = StrFormat("{{NAME}} - {{INTERVAL}} {{DATE}} Open %g, Hi %g, Lo %g, Close %g (%.1f%%) {{VALUES}}", O, H, L, C, SelectedValue(ROC(C, 1))));

Plot(Close, "Close", ParamColor("Color", colorDefault), styleNoTitle | ParamStyle("Style") | GetPriceStyle());

_SECTION_END();
```
