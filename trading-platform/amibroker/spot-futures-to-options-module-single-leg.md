# Spot/Futures to Options Module (Single Leg)

This tutorial provides instructions on how to utilize simple buy and sell trading signals in Spot/Future charts to place option orders (including ATM, ITM, and OTM options) on the OpenAlgo platform. Implementing this system can help traders mitigate risk, particularly in the event of unexpected market movements that may result in significant losses. Additionally, implementing ATM or ITM option buying strategies, as opposed to futures, may help to reduce the risk of extreme black swan events and significantly mitigate the potential for large gap up or gap down risks when carrying forward positions.

**Features of the Options Execution Module**

1\)Simple Drag and Drop Module on top of any Amibroker trading strategy with the proper buy,sell,short,cover defined variables.\
2\)Place Smart Option Orders to intelligent send orders by manipulating the current existing positions.\
3\)Option Strike calculation at Amibroker end (Trades can configure the Underlying symbol as Spot./Futures) based on their trading requirement) accordingly, options strikes will be calculated.

Internet Functions Method (Modern)

```clike
/* 
OpenAlgo - Modern Spot/Futures to Options Trading Module 
Created By: Rajandran R (Founder - Marketcalls / Creator OpenAlgo) 
Created On: 23 Dec 2024 
Website: www.marketcalls.in / www.openalgo.in 
*/

_SECTION_BEGIN("OpenAlgo - Spot/Futures to Options Module");

// Parameter Definitions
RequestTimedRefresh(1, False);
EnableTextOutput(False);

apikey = ParamStr("OpenAlgo API Key", "******");
strategy = ParamStr("Strategy Name", "Test Strategy");
spot = ParamList("Spot Symbol", "NIFTY|BANKNIFTY|FINNIFTY|SENSEX|CRUDEOILM");
expiry = ParamStr("Expiry Date", "17SEP25");
exchange = ParamList("Exchange", "NFO|BFO|MCX", 0);
symbol = ParamStr("Underlying Symbol", "NIFTY");   //Amibroker Symbols Spot/Futures (Calculation of Strike Price)
iInterval = Param("Strike Interval", 50, 1, 10000, 1);
StrikeCalculation = ParamList("Strike Calculation", "PREVOPEN|PREVCLOSE|TODAYSOPEN", 0);
LotSize = Param("Lot Size", 75, 1, 10000, 1);
offsetCE = Param("CE Offset", 0, -40, 40, 1);  //0 - ATM Options,  +4 - 4 strike OTM , -2 = 2 strike wide ITM options
offsetPE = Param("PE Offset", 0, -40, 40, 1);
pricetype = ParamList("Order Type", "MARKET", 0);
product = ParamList("Product", "MIS|NRML", 1);
tradetype = ParamList("Option Trade Type", "BUY|SELL", 0);  //Option Buyer - Option Seller
quantity = Param("Quantity (Lot Size)", 1, 0, 10000) * LotSize;
host = ParamStr("Host", "http://127.0.0.1:5000");
ver = ParamStr("API Version", "v1");
VoiceAlert = ParamList("Voice Alert", "Disable|Enable", 1);
EntryDelay = Param("Entry Delay", 0, 0, 1, 1);
ExitDelay = Param("Exit Delay", 0, 0, 1, 1);
EnableAlgo = ParamList("AlgoStatus", "Disable|Enable|LongOnly|ShortOnly", 0);

SetChartOptions(0, chartShowArrows | chartShowDates);
_N(Title = StrFormat("{{NAME}} - {{INTERVAL}} {{DATE}} Open %g, Hi %g, Lo %g, Close %g (%.1f%%) {{VALUES}}", O, H, L, C, SelectedValue(ROC(C, 1))));

bridgeurl = host + "/api/" + ver;

static_name_ = Name()+GetChartID()+interval(2)+strategy;
static_name_algo = static_name_+interval(2)+strategy+"algostatus";


//OpenAlgo Dashboard

GfxSelectFont( "BOOK ANTIQUA", 14, 100 );
GfxSetBkMode( 1 );
if(EnableAlgo == "Enable")
{
AlgoStatus = "Algo Enabled";
GfxSetTextColor( colorGreen ); 
GfxTextOut( "Algostatus : "+AlgoStatus , 20, 40); 
if(Nz(StaticVarGet(static_name_algo),0)!=1)
{
_TRACE("Algo Status : Enabled");
StaticVarSet(static_name_algo, 1);
}
}
if(EnableAlgo == "Disable")
{
AlgoStatus = "Algo Disabled";
GfxSetTextColor( colorRed ); 
GfxTextOut( "Algostatus : "+AlgoStatus , 20, 40); 
if(Nz(StaticVarGet(static_name_algo),0)!=0)
{
_TRACE("Algo Status : Disabled");
StaticVarSet(static_name_algo, 0);
}
}

// Strike Price Calculations
if (StrikeCalculation == "PREVOPEN") {
    SetForeign(symbol);
    spotC = Ref(Open, -1);
    RestorePriceArrays();
} else if (StrikeCalculation == "PREVCLOSE") {
    SetForeign(symbol);
    spotC = Ref(Close, -1);
    RestorePriceArrays();
} else if (StrikeCalculation == "TODAYSOPEN") {
    SetForeign(symbol);
    spotC = TimeFrameGetPrice("O", inDaily);
    RestorePriceArrays();
}


//Maintain Array to Store ATM Strikes for each and every bar
strike = IIf(spotC % iInterval > iInterval/2, spotC - (spotC%iInterval) + iInterval,
			spotC - (spotC%iInterval));
			
//Entry Strikes	

		
strikeCE = strike + (offsetCE * iInterval);
strikePE = strike - (offsetPE * iInterval);

buycontinue = Flip(Buy,Sell);
shortcontinue  = Flip(Short,Cover);

printf("\n Spot Price :"+spotC);
printf("\n Strike CE :"+strikeCE);
printf("\n Strike PE :"+strikePE);


global ExitStrikeCE;
global ExitStrikePE;
global entryoptions;
global exitoptons;

// Trading Variables
AlgoBuy = LastValue(Ref(Buy, -EntryDelay));
AlgoSell = LastValue(Ref(Sell, -ExitDelay));
AlgoShort = LastValue(Ref(Short, -EntryDelay));
AlgoCover = LastValue(Ref(Cover, -ExitDelay));

//Exit Strikes
if(tradetype=="BUY")
{
ExitStrikeCE = ValueWhen(Ref(Buy,-Entrydelay),strikeCE); 
ExitStrikePE = ValueWhen(Ref(Short,-Entrydelay),strikePE);



entryoptions = WriteIf(AlgoBuy,spot+expiry+LastValue(strikeCE)+"CE", WriteIf(AlgoShort,spot+expiry+LastValue(strikePE)+"PE",""));
exitoptions = WriteIf(AlgoSell,spot+expiry+LastValue(ExitStrikeCE)+"CE", WriteIf(AlgoCover,spot+expiry+LastValue(ExitStrikePE)+"PE",""));


}
if(tradetype=="SELL")
{
ExitStrikeCE = ValueWhen(Ref(Short,-Entrydelay),strikeCE); 
ExitStrikePE = ValueWhen(Ref(Buy,-Entrydelay),strikePE);



entryoptions = WriteIf(AlgoBuy,spot+expiry+LastValue(strikePE)+"PE", WriteIf(AlgoShort,spot+expiry+LastValue(strikeCE)+"CE",""));
exitoptions = WriteIf(AlgoSell,spot+expiry+LastValue(ExitStrikeCE)+"PE", WriteIf(AlgoCover,spot+expiry+LastValue(ExitStrikeCE)+"CE",""));

}

printf("\n\n\nEntry Symbol : "+entryoptions);
printf("\nExit Symbol : "+exitoptions);




// Define Functions for Order Placement and Exit
function PlaceOrder(action, optionType, qty) {
    postData = "{\"apikey\": \"" + apikey + "\", " +
               "\"strategy\": \"" + strategy + "\", " +
               "\"symbol\": \"" + entryoptions + "\", " +
               "\"action\": \"" + action + "\", " +
               "\"exchange\": \"" + exchange + "\", " +
               "\"pricetype\": \"" + pricetype + "\", " +
               "\"product\": \"" + product + "\", " +
               "\"quantity\": \"" + qty + "\"}";
    headers = "Content-Type: application/json\r\nAccept-Encoding: gzip, deflate\r\n";
    InternetSetHeaders(headers);
    ih = InternetPostRequest(bridgeurl + "/placeorder", postData);

    if (ih) {
        response = "";
        while ((line = InternetReadString(ih)) != "") {
            response += line;
        }
        _TRACE("PlaceOrder Request : "+postData);
        _TRACEF("PlaceOrder Response: %s", response);
        if (VoiceAlert == "Enable") Say(action + " Order Placed.");
        InternetClose(ih);
    } else {
        _TRACE("Failed to place order.");
    }
}

function ExitOrder(action, optionType) {
    postData = "{\"apikey\": \"" + apikey + "\", " +
               "\"strategy\": \"" + strategy + "\", " +
               "\"symbol\": \"" + exitoptions + "\", " +
               "\"action\": \"" + action + "\", " +
               "\"exchange\": \"" + exchange + "\", " +
               "\"pricetype\": \"" + pricetype + "\", " +
               "\"product\": \"" + product + "\", " +
               "\"position_size\": \"" + "0" + "\", " +
               "\"quantity\": \"0\"}";
    headers = "Content-Type: application/json\r\nAccept-Encoding: gzip, deflate\r\n";
    InternetSetHeaders(headers);
    ih = InternetPostRequest(bridgeurl + "/placesmartorder", postData);

    if (ih) {
        response = "";
        while ((line = InternetReadString(ih)) != "") {
            response += line;
        }
        _TRACE("PlaceOrder Request : "+postData);
        _TRACEF("ExitOrder Response: %s", response);
        if (VoiceAlert == "Enable") Say(action + " Exit Order Placed.");
        InternetClose(ih);
    } else {
        _TRACE("Failed to exit order.");
    }
}


if (EnableAlgo != "Disable") {
    lasttime = StrFormat("%0.f", LastValue(BarIndex()));

    SetChartBkColor(colorDarkGrey);

    if (EnableAlgo == "Enable" OR EnableAlgo == "LongOnly" OR EnableAlgo == "ShortOnly") {
        if (AlgoBuy == True AND AlgoCover == True AND StaticVarGet(static_name_ + "buyCoverAlgo") == 0 AND StaticVarGetText(static_name_ + "buyCoverAlgo_barvalue") != lasttime) {
            if (tradetype == "BUY") {
                // Long Call and Exit Long Put
                ExitOrder("SELL", "PE");
                PlaceOrder("BUY", "CE", quantity);
            } else if (tradetype == "SELL") {
                // Short Put and Exit Short Call
                ExitOrder("BUY", "CE");
                PlaceOrder("SELL", "PE", quantity);
            }
            _TRACE("Buy Cover Order Triggered.");
            StaticVarSet(static_name_ + "buyCoverAlgo", 1);
            StaticVarSetText(static_name_ + "buyCoverAlgo_barvalue", lasttime);
        } else if (AlgoBuy != True OR AlgoCover != True) {
            StaticVarSet(static_name_ + "buyCoverAlgo", 0);
            StaticVarSetText(static_name_ + "buyCoverAlgo_barvalue", "");
        }

        if (AlgoBuy == True AND AlgoCover != True AND StaticVarGet(static_name_ + "buyAlgo") == 0 AND StaticVarGetText(static_name_ + "buyAlgo_barvalue") != lasttime) {
            if (tradetype == "BUY") {
                PlaceOrder("BUY", "CE", quantity);
            } else if (tradetype == "SELL") {
                PlaceOrder("SELL", "PE", quantity);
            }
            _TRACE("Buy Order Triggered.");
            StaticVarSet(static_name_ + "buyAlgo", 1);
            StaticVarSetText(static_name_ + "buyAlgo_barvalue", lasttime);
        } else if (AlgoBuy != True) {
            StaticVarSet(static_name_ + "buyAlgo", 0);
            StaticVarSetText(static_name_ + "buyAlgo_barvalue", "");
        }

        if (AlgoSell == True AND AlgoShort != True AND StaticVarGet(static_name_ + "sellAlgo") == 0 AND StaticVarGetText(static_name_ + "sellAlgo_barvalue") != lasttime) {
            if (tradetype == "BUY") {
                ExitOrder("SELL", "CE");
            } else if (tradetype == "SELL") {
                ExitOrder("BUY", "PE");
            }
            _TRACE("Sell Order Triggered.");
            StaticVarSet(static_name_ + "sellAlgo", 1);
            StaticVarSetText(static_name_ + "sellAlgo_barvalue", lasttime);
        } else if (AlgoSell != True) {
            StaticVarSet(static_name_ + "sellAlgo", 0);
            StaticVarSetText(static_name_ + "sellAlgo_barvalue", "");
        }

        if (AlgoShort == True AND AlgoSell == True AND StaticVarGet(static_name_ + "ShortSellAlgo") == 0 AND StaticVarGetText(static_name_ + "ShortSellAlgo_barvalue") != lasttime) {
            if (tradetype == "BUY") {
                ExitOrder("SELL", "CE");
                PlaceOrder("BUY", "PE", quantity);
            } else if (tradetype == "SELL") {
                ExitOrder("BUY", "PE");
                PlaceOrder("SELL", "CE", quantity);
            }
            _TRACE("Short Sell Order Triggered.");
            StaticVarSet(static_name_ + "ShortSellAlgo", 1);
            StaticVarSetText(static_name_ + "ShortSellAlgo_barvalue", lasttime);
        } else if (AlgoShort != True OR AlgoSell != True) {
            StaticVarSet(static_name_ + "ShortSellAlgo", 0);
            StaticVarSetText(static_name_ + "ShortSellAlgo_barvalue", "");
        }

        if (AlgoShort == True AND AlgoSell != True AND StaticVarGet(static_name_ + "ShortAlgo") == 0 AND StaticVarGetText(static_name_ + "ShortAlgo_barvalue") != lasttime) {
            if (tradetype == "BUY") {
                PlaceOrder("BUY", "PE", quantity);
            } else if (tradetype == "SELL") {
                PlaceOrder("SELL", "CE", quantity);
            }
            _TRACE("Short Order Triggered.");
            StaticVarSet(static_name_ + "ShortAlgo", 1);
            StaticVarSetText(static_name_ + "ShortAlgo_barvalue", lasttime);
        } else if (AlgoShort != True) {
            StaticVarSet(static_name_ + "ShortAlgo", 0);
            StaticVarSetText(static_name_ + "ShortAlgo_barvalue", "");
        }

        if (AlgoCover == True AND AlgoBuy != True AND StaticVarGet(static_name_ + "CoverAlgo") == 0 AND StaticVarGetText(static_name_ + "CoverAlgo_barvalue") != lasttime) {
            if (tradetype == "BUY") {
                ExitOrder("SELL", "PE");
            } else if (tradetype == "SELL") {
                ExitOrder("BUY", "CE");
            }
            _TRACE("Cover Order Triggered.");
            StaticVarSet(static_name_ + "CoverAlgo", 1);
            StaticVarSetText(static_name_ + "CoverAlgo_barvalue", lasttime);
        } else if (AlgoCover != True) {
            StaticVarSet(static_name_ + "CoverAlgo", 0);
            StaticVarSetText(static_name_ + "CoverAlgo_barvalue", "");
        }
    }
}

_SECTION_END();


```

VB Script Method (Legacy)

```clike

/*
OpenAlgo - Smart Spot/Futures to Options Trading Module
Created By : Rajandran R(Founder - Marketcalls / Creator OpenAlgo )
Created on : 31 MAY 2024.
Website : www.marketcalls.in / www.openalgo.in
*/


_SECTION_BEGIN("Openalgo - Spot/Futures to Options Module");


// Send orders even if Amibroker is minimized or Chart is not active
RequestTimedRefresh(1, False); 
EnableTextOutput(False);


apikey = ParamStr("OpenAlgo API Key", "******");

strategy = ParamStr("Strategy Name", "Test Strategy");

spot = Paramlist("Spot Symbol","NIFTY|BANKNIFTY|FINNIFTY|SENSEX");
expiry = ParamStr("Expiry Date","06JUN24");

exchange = ParamList("Exchange","NFO|BFO|MCX",0); 
Symbol = ParamStr("Underlying Symbol","NIFTY");
iInterval= Param("Strike Interval",50,1,10000,1);
StrikeCalculation = Paramlist("Strike Calculation","PREVOPEN|PREVCLOSE|TODAYSOPEN",0);
LotSize = Param("Lot Size",25,1,10000,1);

offsetCE = Param("CE Offset",0,-40,40,1);
offsetPE = Param("PE Offset",0,-40,40,1);

pricetype = ParamList("Order Type","MARKET",0);
product = ParamList("Product","MIS|NRML",1);
tradetype = ParamList("Option Trade Type","BUY|SELL",0);

quantity = Param("quanity(Lot Size)",1,0,10000)*LotSize;
price = 0; 
disclosed_quantity = 0;
trigger_price = 0;

host = ParamStr("host","http://127.0.0.1:5000");
ver = ParamStr("API Version","v1");


VoiceAlert = ParamList("Voice Alert","Disable|Enable",1);

Entrydelay = Param("Entry Delay",0,0,1,1);
Exitdelay = Param("Exit Delay",0,0,1,1);
EnableAlgo = ParamList("AlgoStatus","Disable|Enable|LongOnly|ShortOnly",0);


bridgeurl = host+"/api/"+ver;
resp = "";


//Static Variables for Order protection

static_name_ = Name()+GetChartID()+interval(2)+strategy;
static_name_algo = Name()+GetChartID()+interval(2)+strategy+"algostatus";


AlgoBuy = lastvalue(Ref(Buy,-Entrydelay));
AlgoSell = lastvalue(Ref(Sell,-Exitdelay));
AlgoShort = lastvalue(Ref(Short,-Entrydelay));
AlgoCover = lastvalue(Ref(Cover,-Exitdelay));

//Plots Dashboard

GfxSelectFont( "BOOK ANTIQUA", 14, 100 );
GfxSetBkMode( 1 );
if (EnableAlgo != "")
{
AlgoStatus = EnableAlgo;
GfxSetTextColor( IIf(EnableAlgo == "Enable", colorGreen, 
					IIf(EnableAlgo == "LongOnly",colorYellow,
						IIf(EnableAlgo == "ShortOnly",colorOrange,colorRed)) ));
GfxTextOut( "Algostatus : "+AlgoStatus , 20, 40);
StaticVarSet(static_name_algo, IIf(EnableAlgo == "Enable", 1,
IIf(EnableAlgo == "LongOnly", 2, IIf(EnableAlgo == "ShortOnly", 3, 0))));
//_TRACE("Algo Status : "+EnableAlgo);
}





//optionCEtype = WriteIf(offsetCE == 0, "ATM CE", WriteIf(offsetCE<0,"ITM"+abs(offsetCE)+" CE","OTM"+abs(offsetCE)+" CE"));
//optionPEtype = WriteIf(offsetPE == 0, "ATM PE", WriteIf(offsetPE<0,"ITM"+abs(offsetPE)+" PE","OTM"+abs(offsetPE)+" PE"));

if(StrikeCalculation=="PREVOPEN")
{
SetForeign(Symbol);
spotC = Ref(OPEN,-1);
RestorePriceArrays();
}

if(StrikeCalculation=="PREVCLOSE")
{
SetForeign(Symbol);
spotC = Ref(Close,-1);
RestorePriceArrays();
}

if(StrikeCalculation=="TODAYSOPEN")
{
SetForeign(Symbol);
spotC = TimeFrameGetPrice("O",inDaily);
RestorePriceArrays();
}

//Maintain Array to Store ATM Strikes for each and every bar
strike = IIf(spotC % iInterval > iInterval/2, spotC - (spotC%iInterval) + iInterval,
			spotC - (spotC%iInterval));
			
//Entry Strikes	
		
strikeCE = strike + (offsetCE * iInterval);
strikePE = strike - (offsetPE * iInterval);

buycontinue = Flip(Buy,Sell);
shortcontinue  = Flip(Short,Cover);


ExitStrikeCE = "";
ExitStrikePE = "";
entryoptions = "";
exitoptons = "";

//Exit Strikes
if(tradetype=="BUY")
{
ExitStrikeCE = ValueWhen(Ref(Buy,-Entrydelay),strikeCE); 
ExitStrikePE = ValueWhen(Ref(Short,-Entrydelay),strikePE);

entryoptions = WriteIf(Buy,spot+expiry+strikeCE+"CE", WriteIf(Short,spot+expiry+strikePE+"PE",""));
exitoptons = WriteIf(sell,spot+expiry+ExitStrikeCE+"CE", WriteIf(cover,spot+expiry+ExitStrikePE+"PE",""));


}
if(tradetype=="SELL")
{
ExitStrikeCE = ValueWhen(Ref(Short,-Entrydelay),strikeCE); 
ExitStrikePE = ValueWhen(Ref(Buy,-Entrydelay),strikePE);

entryoptions = WriteIf(Buy,spot+expiry+strikePE+"PE", WriteIf(Short,spot+expiry+strikeCE+"CE",""));
exitoptons = WriteIf(sell,spot+expiry+ExitStrikePE+"PE", WriteIf(cover,spot+expiry+ExitStrikeCE+"CE",""));

}

printf("\n\n\nEntry Symbol : "+entryoptions);
printf("\nExit Symbol : "+exitoptons);


_SECTION_END();




_SECTION_BEGIN("OpenAlgo Bridge Controls");

EnableScript("VBScript"); 
<%
Public Sub PlaceOrder(action, OptionType, quantity)
	
    Dim oXMLHTTP
    Dim oStream
    Set oXMLHTTP = CreateObject("Msxml2.XMLHTTP")
    ' Define variables with the specified values
    Dim apikey, strategy, symbol , exchange, pricetype, product
    apikey = AFL.Var("apikey")
    strategy = AFL.Var("strategy")
    

	
    
    symbol = AFL.Var("entryoptions")
    
    exchange = AFL.Var("exchange")
    pricetype = AFL.Var("pricetype")
    product = AFL.Var("product")
   
    
    ' Construct the JSON string for the POST message
    Dim jsonRequestBody
    jsonRequestBody = "{""apikey"":""" & apikey & _
    """,""strategy"":""" & strategy & _
    """,""symbol"":""" & symbol & _
    """,""action"":""" & action & _
    """,""exchange"":""" & exchange & _
    """,""pricetype"":""" & pricetype & _
    """,""product"":""" & product & _
    """,""quantity"":""" & quantity & """}"
    
    ' Set the URL
    Dim url
    url = AFL.Var("bridgeurl")&"/placeorder"
    
    ' MsgBox "API Request: " & jsonRequestBody, vbInformation, "API Information"
    
    ' Configure the HTTP request for POST method
    oXMLHTTP.Open "POST", url, False
    oXMLHTTP.setRequestHeader "Content-Type", "application/json"
    oXMLHTTP.setRequestHeader "Cache-Control", "no-cache"
    oXMLHTTP.setRequestHeader "Pragma", "no-cache"
    
    ' Send the request with the JSON body
    oXMLHTTP.Send jsonRequestBody
    
    api_parameters = "Strategy :" & strategy & " Symbol :" & symbol & " Exchange :" & exchange & _
                 " Action :" & action & " Pricetype :" & pricetype & _
                 " Product :" & product & " Quantity:" & quantity & _
                 " api_url :" & url

    
    ' MsgBox "API Request: " & oXMLHTTP.responseText, vbInformation, "API Information"
    
    
    AFL("api_request") = api_parameters  
    AFL("api_response") = oXMLHTTP.responseText
    
    
    ' Optionally, handle the response here
    ' Dim response
    ' response = oXMLHTTP.responseText
    ' Response handling code...
End Sub


Public Sub ExitOrder(action, OptionType)
    Dim oXMLHTTP
    Dim oStream
    Set oXMLHTTP = CreateObject("Msxml2.XMLHTTP")
    ' Define variables with the specified values
    Dim apikey, strategy, symbol , exchange, pricetype, product
    apikey = AFL.Var("apikey")
    strategy = AFL.Var("strategy")
   	
    
	symbol = AFL.Var("exitoptons")
    
    exchange = AFL.Var("exchange")
    pricetype = AFL.Var("pricetype")
    product = AFL.Var("product")
    position_size = "0"
    quantity = "0"
   
    
    ' Construct the JSON string for the POST message
    Dim jsonRequestBody
    jsonRequestBody = "{""apikey"":""" & apikey & _
    """,""strategy"":""" & strategy & _
    """,""symbol"":""" & symbol & _
    """,""action"":""" & action & _
    """,""exchange"":""" & exchange & _
    """,""pricetype"":""" & pricetype & _
    """,""product"":""" & product & _
    """,""quantity"":""" & quantity & _
    """,""position_size"":""" & position_size & """}"
    
    ' Set the URL
    Dim url
    url = AFL.Var("bridgeurl")&"/placesmartorder"
    
    ' Configure the HTTP request for POST method
    oXMLHTTP.Open "POST", url, False
    oXMLHTTP.setRequestHeader "Content-Type", "application/json"
    oXMLHTTP.setRequestHeader "Cache-Control", "no-cache"
    oXMLHTTP.setRequestHeader "Pragma", "no-cache"
    
    ' Send the request with the JSON body
    oXMLHTTP.Send jsonRequestBody
    
    api_parameters = "Strategy :" & strategy & " Symbol :" & symbol & " Exchange :" & exchange & _
                 " Action :" & action & " Pricetype :" & pricetype & _
                 " Product :" & product & " Quantity:" & quantity & _
                 " Position Size :" & position_size & " api_url :" & url

    
    AFL("ex_api_request") = api_parameters  
    AFL("ex_api_response") = oXMLHTTP.responseText
    
    
    ' Optionally, handle the response here
    ' Dim response
    ' response = oXMLHTTP.responseText
    ' Response handling code...
End Sub

%>

openalgo = GetScriptObject();



_SECTION_END();


if(EnableAlgo != "Disable")
    {
        lasttime = StrFormat("%0.f",LastValue(BarIndex()));
        
       
        
        SetChartBkColor(colorDarkGrey);
        if(EnableAlgo == "Enable")
        {   
			
			
            if (AlgoBuy==True AND AlgoCover == True AND StaticVarGet(static_name_+"buyCoverAlgo")==0 AND StaticVarGetText(static_name_+"buyCoverAlgo_barvalue") != lasttime )
            {
            
				if(tradetype=="BUY")
				{
				
				
				
				//Long Call and Exit Long Put Option
				openalgo.ExitOrder("SELL", "PE"); 
				openalgo.PlaceOrder("BUY", "CE",quantity);

				
				
				}
				
				if(tradetype=="SELL")
				{
				//Short Put and Exit Short Call Option
				openalgo.ExitOrder("BUY" , "CE"); 
				openalgo.PlaceOrder("SELL" ,"PE",quantity);
				
				
				
				}
				
				_TRACE("Exit API Request : "+ex_api_request);
				_TRACE("Exit API Response : "+ex_api_response);
				_TRACE("API Request : "+api_request);
				_TRACE("API Response : "+api_response);
                StaticVarSetText(static_name_+"buyCoverAlgo_barvalue",lasttime);  
                StaticVarSet(static_name_+"buyCoverAlgo",1); //Algo Order was triggered, no more order on this bar
                
        
            }
            else if ((AlgoBuy != True OR AlgoCover != True))
            {   
                StaticVarSet(static_name_+"buyCoverAlgo",0);
                StaticVarSetText(static_name_+"buyCoverAlgo_barvalue","");
            }
            
            if (AlgoBuy==True AND AlgoCover != True AND StaticVarGet(static_name_+"buyAlgo")==0 AND StaticVarGetText(static_name_+"buyAlgo_barvalue") != lasttime)
            {
            
				if(tradetype=="BUY")
				{
				//Long Call and Exit Long Put Option
				openalgo.PlaceOrder("BUY","CE",quantity);

				
				}
				
				if(tradetype=="SELL")
				{
				//Short Put
				openalgo.PlaceOrder("SELL","PE",quantity);
				
					
				}
				
				_TRACE("API Request : "+api_request);
				_TRACE("API Response : "+api_response);
                StaticVarSetText(static_name_+"buyAlgo_barvalue",lasttime); 
                StaticVarSet(static_name_+"buyAlgo",1); //Algo Order was triggered, no more order on this bar
                
            }
            else if (AlgoBuy != True)
            {   
                StaticVarSet(static_name_+"buyAlgo",0);
                StaticVarSetText(static_name_+"buyAlgo_barvalue","");
                
            }
            if (AlgoSell==true AND AlgoShort != True AND StaticVarGet(static_name_+"sellAlgo")==0 AND StaticVarGetText(static_name_+"sellAlgo_barvalue") != lasttime)
            {     
				
				if(tradetype=="BUY")
				{
				//Exit Long Call Option
				
				openalgo.ExitOrder("SELL","CE"); 
				
				}
				
				if(tradetype=="SELL")
				{
				//Exit Short Put Option
				openalgo.ExitOrder("BUY","PE"); 
				
				}
                
                _TRACE("Exit API Request : "+ex_api_request);
				_TRACE("Exit API Response : "+ex_api_response);

                StaticVarSetText(static_name_+"sellAlgo_barvalue",lasttime);
                StaticVarSet(static_name_+"sellAlgo",1); //Algo Order was triggered, no more order on this bar
                
            }
            else if (AlgoSell != True )
            {   
                StaticVarSet(static_name_+"sellAlgo",0);
                StaticVarSetText(static_name_+"sellAlgo_barvalue","");
            }
            if (AlgoShort==True AND AlgoSell==True AND  StaticVarGet(static_name_+"ShortSellAlgo")==0 AND StaticVarGetText(static_name_+"ShortSellAlgo_barvalue") != lasttime)
            {
            
				if(tradetype=="BUY")
				{
				//Long Put and Exit Long Call Option
				openalgo.ExitOrder("SELL","CE"); 
				openalgo.PlaceOrder("BUY","PE",quantity);
				
				
				}
				
				if(tradetype=="SELL")
				{
				//Short Call and Exit Short Put Option
				openalgo.ExitOrder("BUY","PE");
				openalgo.PlaceOrder("SELL","CE",quantity);
				 
				
				}
				
				_TRACE("Exit API Request : "+ex_api_request);
				_TRACE("Exit API Response : "+ex_api_response);
				_TRACE("API Request : "+api_request);
				_TRACE("API Response : "+api_response);
				
                StaticVarSetText(static_name_+"ShortsellAlgo_barvalue",lasttime);
                StaticVarSet(static_name_+"ShortSellAlgo",1); //Algo Order was triggered, no more order on this bar
                
            }
            else if ((AlgoShort != True OR AlgoSell != True))
            {   
                StaticVarSet(static_name_+"ShortSellAlgo",0);
                StaticVarSetText(static_name_+"ShortsellAlgo_barvalue","");
            }
                
            if (AlgoShort==True  AND  AlgoSell != True AND StaticVarGet(static_name_+"ShortAlgo")==0 AND  StaticVarGetText(static_name_+"ShortAlgo_barvalue") != lasttime)
            {
				if(tradetype=="BUY")
				{
				//Long Put
				openalgo.PlaceOrder("BUY","PE",quantity);
				
				
				}
				
				if(tradetype=="SELL")
				{
				//Short Call
				openalgo.PlaceOrder("SELL","CE",quantity);
							
				}
				
				

				_TRACE("API Request : "+api_request);
				_TRACE("API Response : "+api_response);
                StaticVarSetText(static_name_+"ShortAlgo_barvalue",lasttime); 
                StaticVarSet(static_name_+"ShortAlgo",1); //Algo Order was triggered, no more order on this bar
                
            }
            else if (AlgoShort != True )
            {   
                StaticVarSet(static_name_+"ShortAlgo",0);
                StaticVarSetText(static_name_+"ShortAlgo_barvalue","");
            }
            if (AlgoCover==true AND AlgoBuy != True AND StaticVarGet(static_name_+"CoverAlgo")==0 AND StaticVarGetText(static_name_+"CoverAlgo_barvalue") != lasttime)
            {
				
				if(tradetype=="BUY")
				{
				//Exit Long Put Option
				
				openalgo.ExitOrder("SELL","PE"); 
				
				}
				
				if(tradetype=="SELL")
				{
				//Exit Short Call Option
				openalgo.ExitOrder("BUY","CE"); 
				
				}
               
               
				_TRACE("Exit API Request : "+ex_api_request);
				_TRACE("Exit API Response : "+ex_api_response);

				
                StaticVarSetText(static_name_+"CoverAlgo_barvalue",lasttime); 
                StaticVarSet(static_name_+"CoverAlgo",1); //Algo Order was triggered, no more order on this bar
                
            }
            else if (AlgoCover != True )
            {   
                StaticVarSet(static_name_+"CoverAlgo",0);
                StaticVarSetText(static_name_+"CoverAlgo_barvalue","");
            }
        }
        
      else if(EnableAlgo == "LongOnly")
        {
        
			 if (AlgoBuy==True AND StaticVarGet(static_name_+"buyAlgo")==0 AND StaticVarGetText(static_name_+"buyAlgo_barvalue") != lasttime)
            {
            
				if(tradetype=="BUY")
				{
				//Long Call and Exit Long Put Option
				openalgo.PlaceOrder("BUY","CE",quantity);
				
				
				}
				
				if(tradetype=="SELL")
				{
				//Short Put
				openalgo.PlaceOrder("SELL","PE",quantity);
					
				}
				

				_TRACE("API Request : "+api_request);
				_TRACE("API Response : "+api_response);
                StaticVarSetText(static_name_+"buyAlgo_barvalue",lasttime);
                StaticVarSet(static_name_+"buyAlgo",1); //Algo Order was triggered, no more order on this bar
                
            }
            
            
            else if (AlgoBuy != True )
            {  
                StaticVarSet(static_name_+"buyAlgo",0);
                StaticVarSetText(static_name_+"buyAlgo_barvalue","");
            
            }
            
             if (AlgoSell==True AND StaticVarGet(static_name_+"sellAlgo")==0 AND StaticVarGetText(static_name_+"sellAlgo_barvalue") != lasttime)
            { 
            
				if(tradetype=="BUY")
				{
				//Exit Long Call Option
				
				openalgo.ExitOrder("SELL","CE"); 
				
				}
				
				if(tradetype=="SELL")
				{
				//Exit Short Put Option
				openalgo.ExitOrder("BUY","PE"); 
				
				}
				
				_TRACE("Exit API Request : "+ex_api_request);
				_TRACE("Exit API Response : "+ex_api_response);

                StaticVarSet(static_name_+"sellAlgo",1);
                StaticVarSetText(static_name_+"sellAlgo_barvalue",lasttime);
            }
            else if (AlgoSell != True )
            {  
                StaticVarSet(static_name_+"sellAlgo",0);
                StaticVarSetText(static_name_+"sellAlgo_barvalue","");
            
            }
            
        } 
        else if(EnableAlgo == "ShortOnly")
        {
            if (AlgoShort==True AND StaticVarGet(static_name_+"ShortAlgo")==0 AND StaticVarGetText(static_name_+"ShortAlgo_barvalue") != lasttime)
            {
				if(tradetype=="BUY")
				{
				//Long Put
				openalgo.PlaceOrder("BUY","PE",quantity);
				
				
				}
				
				if(tradetype=="SELL")
				{
				//Short Call
				openalgo.PlaceOrder("SELL","CE",quantity);
							
				}
				

				_TRACE("API Request : "+api_request);
				_TRACE("API Response : "+api_response);
                StaticVarSetText(static_name_+"ShortAlgo_barvalue",lasttime); 
                StaticVarSet(static_name_+"ShortAlgo",1); //Algo Order was triggered, no more order on this bar
                
            }
            else if (AlgoShort != True )
            {   
                StaticVarSet(static_name_+"ShortAlgo",0);
                StaticVarSetText(static_name_+"ShortAlgo_barvalue","");
            }
            if (AlgoCover==true AND StaticVarGet(static_name_+"CoverAlgo")==0 AND StaticVarGetText(static_name_+"CoverAlgo_barvalue") != lasttime)
            {
            
				if(tradetype=="BUY")
				{
				//Exit Long Put Option
				
				openalgo.ExitOrder("SELL","PE"); 
				
				}
				
				if(tradetype=="SELL")
				{
				//Exit Short Call Option
				openalgo.ExitOrder("BUY","CE"); 
				
				}
               
               
				_TRACE("Exit API Request : "+ex_api_request);
				_TRACE("Exit API Response : "+ex_api_response);

				
                StaticVarSetText(static_name_+"CoverAlgo_barvalue",lasttime); 
                StaticVarSet(static_name_+"CoverAlgo",1); //Algo Order was triggered, no more order on this bar
                
            }
            else if (AlgoCover != True)
            {   
                StaticVarSet(static_name_+"CoverAlgo",0);
                StaticVarSetText(static_name_+"CoverAlgo_barvalue","");
            }
        } 
        
    }
    
  
_SECTION_END();

```
