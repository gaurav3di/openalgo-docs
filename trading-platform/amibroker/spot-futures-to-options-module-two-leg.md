# Spot/Futures to Options Module (Two Leg)

**What is a Two-Legged Options Strategy**?

A two-legged options trading strategy involves buying or selling two options at the same time, typically with different strike prices or expiration dates. The strategy is designed to take advantage of the price difference between the two options and can be used for a variety of purposes, such as hedging, speculation, or income generation.

**Features of the Two-Legged Options Execution Module**

1\)Simple Drag and Drop Module on top of any Amibroker trading strategy with the proper buy, sell, short, and cover defined variables.\
2\)Configure various styles of two-legged options execution strategies. Supports 9 types of two-legged options trading strategies.\
3\)Configure separate options trading strategies for long-entry and short-entry signals in spot/futures charts\
4\)Place Smart Option Orders to intelligent send orders by manipulating the current existing positions.\
5\)Option Strike calculation at Amibroker end (Trades can configure the Underlying symbol as Spot./Futures) based on their trading requirement) accordingly, options strikes will be calculated.

```clike

/*
OpenAlgo - Smart Spot/Futures to Two Leg Options Trading Module

Supported Two Leg Strategies
Strategies that be built using this module
1)Credit Spread
2)Debit Spread
3)Straddle
4)Strangle
5)Synthetic Futures
6)Diagonal spread
7)Calendar Spread
8)Ratio Spread
9)Ratio Back Spread


Created By : Rajandran R(Founder - Marketcalls / Creator - Openalgo )
Created on : 4 Jun 2024.
Website : www.marketcalls.in / www.openalgo.in
*/




_SECTION_BEGIN("Openalgo - Order Controls");

// Send orders even if Amibroker is minimized or Chart is not active
RequestTimedRefresh(1, False); 
EnableTextOutput(False);


apikey = ParamStr("OpenAlgo API Key", "******");

strategy = ParamStr("Strategy Name", "Test Strategy");



spot = Paramlist("Spot Symbol","NIFTY|BANKNIFTY|FINNIFTY|SENSEX");
expiry_leg1 = ParamStr("Expiry Date1","06JUN24");
expiry_leg2 = ParamStr("Expiry Date2","06JUN24");

exchange = ParamList("Exchange","NFO|BFO",0); 
Symbol = ParamStr("Underlying Symbol(Datafeed Symbol)","NIFTY"); 
iInterval= Param("Strike Interval",50,1,10000,1);
StrikeCalculation = Paramlist("Strike Calculation","PREVOPEN|PREVCLOSE|TODAYSOPEN",0);
LotSize = Param("Lot Size",25,1,10000,1);

quantity_leg1 = Param("quanity1(Lot Size)",1,0,10000)*LotSize;
quantity_leg2 = Param("quanity2(Lot Size)",1,0,10000)*LotSize;

opttype_buyleg1 = ParamList("Option Type1(Buy)","CE|PE",0);
opttype_buyleg2 = ParamList("Option Type2(Buy)","CE|PE",0);

tradetype_buyleg1 = ParamList("Option Trade Type1(Buy)","BUY|SELL",0);
tradetype_buyleg2 = ParamList("Option Trade Type2(Buy)","BUY|SELL",0);


offset_buyleg1 = Param("Offset1(Buy)",0,-40,40,1);
offset_buyleg2 = Param("Offset2(Buy)",0,-40,40,1);

opttype_shortleg1 = ParamList("Option Type1(Short)","CE|PE",0);
opttype_shortleg2 = ParamList("Option Type2(Short)","CE|PE",0);



tradetype_shortleg1 = ParamList("Option Trade Type1(Short)","BUY|SELL",0);
tradetype_shortleg2 = ParamList("Option Trade Type2(Short)","BUY|SELL",0);


offset_shortleg1 = Param("Offset1(Short)",0,-40,40,1);
offset_shortleg2 = Param("Offset2(Short)",0,-40,40,1);

pricetype = ParamList("Order Type","MARKET",0);
product = ParamList("Product","MIS|NRML",1);


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
if(EnableAlgo == "LongOnly")
{
AlgoStatus = "Long Only";
GfxSetTextColor( colorYellow ); 
GfxTextOut( "Algostatus : "+AlgoStatus , 20, 40); 
if(Nz(StaticVarGet(static_name_algo),0)!=2)
{
_TRACE("Algo Status : Long Only");
StaticVarSet(static_name_algo, 2);
}
}
if(EnableAlgo == "ShortOnly")
{
AlgoStatus = "Short Only";
GfxSetTextColor( colorYellow ); 
GfxTextOut( "Algostatus : "+AlgoStatus , 20, 40); 
if(Nz(StaticVarGet(static_name_algo),0)!=3)
{
_TRACE("Algo Status : Short Only");
StaticVarSet(static_name_algo, 3);
}
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
			
//Buy Signal Entry Strikes	

if(opttype_buyleg1=="CE")
{	
strike_buyleg1 = strike + (offset_buyleg1 * iInterval);
}
if(opttype_buyleg1=="PE")
{	
strike_buyleg1 = strike - (offset_buyleg1 * iInterval);
}
if(opttype_buyleg2=="CE")
{	
strike_buyleg2 = strike + (offset_buyleg2 * iInterval);
}
if(opttype_buyleg2=="PE")
{	
strike_buyleg2 = strike - (offset_buyleg2 * iInterval);
}

//Short Signal Entry Strikes	

if(opttype_shortleg1=="CE")
{	
strike_shortleg1 = strike + (offset_shortleg1 * iInterval);
}
if(opttype_shortleg1=="PE")
{	
strike_shortleg1 = strike - (offset_shortleg1 * iInterval);
}
if(opttype_shortleg2=="CE")
{	
strike_shortleg2 = strike + (offset_shortleg2 * iInterval);
}
if(opttype_shortleg2=="PE")
{	
strike_shortleg2 = strike - (offset_shortleg2 * iInterval);
}


buycontinue = Flip(Buy,Sell);
shortcontinue  = Flip(Short,Cover);

entrysymbol_leg1 = "";
entrysymbol_leg2 = "";


exitsymbol_leg1 = "";
exitsymbol_leg2 = "";


ExitStrike_buyleg1 = ValueWhen(Ref(Buy,-Entrydelay),strike_buyleg1);
ExitStrike_buyleg2 = ValueWhen(Ref(Buy,-Entrydelay),strike_buyleg2);

ExitStrike_shortleg1 = ValueWhen(Ref(Short,-Entrydelay),strike_shortleg1);
ExitStrike_shortleg2 = ValueWhen(Ref(Short,-Entrydelay),strike_shortleg2);

//-------------prepare the enty and exit symbols for two legged option strategy



entrysymbol_leg1 = WriteIf(Buy,spot+expiry_leg1+strike_buyleg1+opttype_buyleg1, WriteIf(Short,spot+expiry_leg1+strike_shortleg1+opttype_shortleg1,""));
exitsymbol_leg1 = WriteIf(sell,spot+expiry_leg1+ExitStrike_buyleg1+opttype_buyleg1, WriteIf(cover,spot+expiry_leg1+ExitStrike_shortleg1+opttype_shortleg1,""));

entrysymbol_leg2 = WriteIf(Buy,spot+expiry_leg2+strike_buyleg2+opttype_buyleg2, WriteIf(Short,spot+expiry_leg2+strike_shortleg2+opttype_shortleg2,""));
exitsymbol_leg2 = WriteIf(sell,spot+expiry_leg2+ExitStrike_buyleg2+opttype_buyleg2, WriteIf(cover,spot+expiry_leg2+ExitStrike_shortleg2+opttype_shortleg2,""));



printf("\n\n\nEntry Leg1 Symbol : "+entrysymbol_leg1);
printf("\nExit Leg1 Symbol : "+exitsymbol_leg1);

printf("\n\n\nEntry Leg2 Symbol : "+entrysymbol_leg2);
printf("\nExit Leg2 Symbol : "+exitsymbol_leg2);



_SECTION_END();




//Buy and Sell Order Functions
//signaltype = "buy" OR "short
//leg = "leg1","leg2"


//Squareoff Function to Exit Open Positions

//signaltype = "buy" OR "short
//leg = "leg1","leg2"





_SECTION_BEGIN("OpenAlgo Bridge Controls");

EnableScript("VBScript"); 
<%
Public Sub PlaceOrder(action, OptionType, quantity,expiry,signaltype,leg)
	
    Dim oXMLHTTP
    Dim oStream
    Set oXMLHTTP = CreateObject("Msxml2.XMLHTTP")
    ' Define variables with the specified values
    Dim apikey, strategy, symbol , exchange, pricetype, product
    apikey = AFL.Var("apikey")
    strategy = AFL.Var("strategy")
    

	
    
    If leg = "leg1"  Then
		symbol = AFL.Var("entrysymbol_leg1")
	elseIf leg = "leg2" Then
			symbol = AFL.Var("entrysymbol_leg2")
	End If


   
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


Public Sub ExitOrder(action, OptionType,expiry,signaltype,leg)
    Dim oXMLHTTP
    Dim oStream
    Set oXMLHTTP = CreateObject("Msxml2.XMLHTTP")
    ' Define variables with the specified values
    Dim apikey, strategy, symbol , exchange, pricetype, product
    apikey = AFL.Var("apikey")
    strategy = AFL.Var("strategy")
   	
    
    If leg = "leg1"  Then
		symbol = AFL.Var("exitsymbol_leg1")
	elseIf leg = "leg2" Then
			symbol = AFL.Var("exitsymbol_leg2")
	End If
    
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




//Configure Trace Logs

if(EnableAlgo != "Disable")
    {
        lasttime = StrFormat("%0.f",LastValue(BarIndex()));
        
        SetChartBkColor(colorDarkGrey);
        if(EnableAlgo == "Enable")
        {   
            if (AlgoBuy==True AND AlgoCover == True AND StaticVarGet(static_name_+"buyCoverAlgo")==0 AND StaticVarGetText(static_name_+"buyCoverAlgo_barvalue") != lasttime )
            {
            
	//ExitOrder(action,OptionType,orderqty,position_size,expiry,signaltype,leg)
	//leg = "leg1","leg2"
				
				//Cover Signal
				if(tradetype_shortleg1=="SELL")
				{
				openalgo.ExitOrder(tradetype_shortleg1,opttype_shortleg1,expiry_leg1,"short","leg1"); 
				_TRACE("Exit API Request leg1: "+ex_api_request);
				_TRACE("Exit API Response leg1: "+ex_api_response);
				openalgo.ExitOrder(tradetype_shortleg2,opttype_shortleg2,expiry_leg2,"short","leg2"); 
				_TRACE("Exit API Request leg2: "+ex_api_request);
				_TRACE("Exit API Response leg2: "+ex_api_response);
				}
				else
				{
				openalgo.ExitOrder(tradetype_shortleg2,opttype_shortleg2,expiry_leg2,"short","leg2");
				_TRACE("Exit API Request leg2: "+ex_api_request);
				_TRACE("Exit API Response leg2: "+ex_api_response);
				openalgo.ExitOrder(tradetype_shortleg1,opttype_shortleg1,expiry_leg1,"short","leg1"); 
				_TRACE("Exit API Request leg1: "+ex_api_request);
				_TRACE("Exit API Response leg1: "+ex_api_response); 
				}
				//Buy Signal
				if(tradetype_buyleg1=="BUY")
				{
				openalgo.PlaceOrder(tradetype_buyleg1,opttype_buyleg1,quantity_leg1,expiry_leg1,"buy","leg1");
				_TRACE("API Request leg1: "+api_request);
				_TRACE("API Response leg1: "+api_response);
				openalgo.PlaceOrder(tradetype_buyleg2,opttype_buyleg2,quantity_leg2,expiry_leg2,"buy","leg2");
				_TRACE("API Request leg2: "+api_request);
				_TRACE("API Response leg2: "+api_response);
				}
				else
				{
				openalgo.PlaceOrder(tradetype_buyleg2,opttype_buyleg2,quantity_leg2,expiry_leg2,"buy","leg2");
				_TRACE("API Request leg2: "+api_request);
				_TRACE("API Response leg2: "+api_response);
				openalgo.PlaceOrder(tradetype_buyleg1,opttype_buyleg1,quantity_leg1,expiry_leg1,"buy","leg1");
				_TRACE("API Request leg1: "+api_request);
				_TRACE("API Response leg1: "+api_response);
				}
					
				
				
				
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
				
				//Buy Signal
				if(tradetype_buyleg1=="BUY")
				{
				openalgo.PlaceOrder(tradetype_buyleg1,opttype_buyleg1,quantity_leg1,expiry_leg1,"buy","leg1");
				_TRACE("API Request leg1: "+api_request);
				_TRACE("API Response leg1: "+api_response);
				openalgo.PlaceOrder(tradetype_buyleg2,opttype_buyleg2,quantity_leg2,expiry_leg2,"buy","leg2");
				_TRACE("API Request leg2: "+api_request);
				_TRACE("API Response leg2: "+api_response);
				}
				else
				{
				openalgo.PlaceOrder(tradetype_buyleg2,opttype_buyleg2,quantity_leg2,expiry_leg2,"buy","leg2");
				_TRACE("API Request leg2: "+api_request);
				_TRACE("API Response leg2: "+api_response);
				openalgo.PlaceOrder(tradetype_buyleg1,opttype_buyleg1,quantity_leg1,expiry_leg1,"buy","leg1");
				_TRACE("API Request leg1: "+api_request);
				_TRACE("API Response leg1: "+api_response);
				}
				
				
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
				//Sell Signal
				if(tradetype_buyleg1=="SELL")
				{
				openalgo.ExitOrder(tradetype_buyleg1,opttype_buyleg1,expiry_leg1,"buy","leg1"); 
				_TRACE("Exit API Request leg1: "+ex_api_request);
				_TRACE("Exit API Response leg1: "+ex_api_response); 
				openalgo.ExitOrder(tradetype_buyleg2,opttype_buyleg2,expiry_leg2,"buy","leg2"); 
				_TRACE("Exit API Request leg2: "+ex_api_request);
				_TRACE("Exit API Response leg2: "+ex_api_response); 
				}
				else
				{
				openalgo.ExitOrder(tradetype_buyleg2,opttype_buyleg2,expiry_leg2,"buy","leg2");
				_TRACE("Exit API Request leg2: "+ex_api_request);
				_TRACE("Exit API Response leg2: "+ex_api_response); 
				openalgo.ExitOrder(tradetype_buyleg1,opttype_buyleg1,expiry_leg1,"buy","leg1");  
				_TRACE("Exit API Request leg1: "+ex_api_request);
				_TRACE("Exit API Response leg1: "+ex_api_response); 
				}
                
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
				//Sell Signal
				if(tradetype_buyleg1=="SELL")
				{
				openalgo.ExitOrder(tradetype_buyleg1,opttype_buyleg1,expiry_leg1,"buy","leg1"); 
				_TRACE("Exit API Request leg1: "+ex_api_request);
				_TRACE("Exit API Response leg1: "+ex_api_response); 
				openalgo.ExitOrder(tradetype_buyleg2,opttype_buyleg2,expiry_leg2,"buy","leg2"); 
				_TRACE("Exit API Request leg2: "+ex_api_request);
				_TRACE("Exit API Response leg2: "+ex_api_response); 
				}
				else
				{
				openalgo.ExitOrder(tradetype_buyleg2,opttype_buyleg2,expiry_leg2,"buy","leg2");
				_TRACE("Exit API Request leg2: "+ex_api_request);
				_TRACE("Exit API Response leg2: "+ex_api_response); 
				openalgo.ExitOrder(tradetype_buyleg1,opttype_buyleg1,expiry_leg1,"buy","leg1"); 
				_TRACE("Exit API Request leg1: "+ex_api_request);
				_TRACE("Exit API Response leg1: "+ex_api_response);  
				}
				//Short
				if(tradetype_shortleg1=="BUY")
				{
				openalgo.PlaceOrder(tradetype_shortleg1,opttype_shortleg1,quantity_leg1,expiry_leg1,"short","leg1");
				_TRACE("API Request leg1: "+api_request);
				_TRACE("API Response leg1: "+api_response);
				openalgo.PlaceOrder(tradetype_shortleg2,opttype_shortleg2,quantity_leg2,expiry_leg2,"short","leg2");
				_TRACE("API Request leg2: "+api_request);
				_TRACE("API Response leg2: "+api_response);
				}
				else
				{
				openalgo.PlaceOrder(tradetype_shortleg2,opttype_shortleg2,quantity_leg2,expiry_leg2,"short","leg2");
				_TRACE("API Request leg2: "+api_request);
				_TRACE("API Response leg2: "+api_response);
				openalgo.PlaceOrder(tradetype_shortleg1,opttype_shortleg1,quantity_leg1,expiry_leg1,"short","leg1");
				_TRACE("API Request leg1: "+api_request);
				_TRACE("API Response leg1: "+api_response);
				}
				
				
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
				//Short
				if(tradetype_shortleg1=="BUY")
				{
				openalgo.PlaceOrder(tradetype_shortleg1,opttype_shortleg1,quantity_leg1,expiry_leg1,"short","leg1");
				_TRACE("API Request leg1: "+api_request);
				_TRACE("API Response leg1: "+api_response);
				openalgo.PlaceOrder(tradetype_shortleg2,opttype_shortleg2,quantity_leg2,expiry_leg2,"short","leg2");
				_TRACE("API Request leg2: "+api_request);
				_TRACE("API Response leg2: "+api_response);
				}
				else
				{
				openalgo.PlaceOrder(tradetype_shortleg2,opttype_shortleg2,quantity_leg2,expiry_leg2,"short","leg2");
				_TRACE("API Request leg2: "+api_request);
				_TRACE("API Response leg2: "+api_response);
				openalgo.PlaceOrder(tradetype_shortleg1,opttype_shortleg1,quantity_leg1,expiry_leg1,"short","leg1");
				_TRACE("API Request leg1: "+api_request);
				_TRACE("API Response leg1: "+api_response);
				}
				
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
				
				//Cover Signal
				if(tradetype_shortleg1=="SELL")
				{
				openalgo.ExitOrder(tradetype_shortleg1,opttype_shortleg1,expiry_leg1,"short","leg1"); 
				_TRACE("Exit API Request leg1: "+ex_api_request);
				_TRACE("Exit API Response leg1: "+ex_api_response); 
				openalgo.ExitOrder(tradetype_shortleg2,opttype_shortleg2,expiry_leg2,"short","leg2"); 
				_TRACE("Exit API Request leg2: "+ex_api_request);
				_TRACE("Exit API Response leg2: "+ex_api_response); 
				}
				else
				{
				openalgo.ExitOrder(tradetype_shortleg2,opttype_shortleg2,expiry_leg2,"short","leg2");
				_TRACE("Exit API Request leg2: "+ex_api_request);
				_TRACE("Exit API Response leg2: "+ex_api_response); 
				openalgo.ExitOrder(tradetype_shortleg1,opttype_shortleg1,expiry_leg1,"short","leg1");  
				_TRACE("Exit API Request leg1: "+ex_api_request);
				_TRACE("Exit API Response leg1: "+ex_api_response); 
				}
               
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
            
				//Buy Signal
				if(tradetype_buyleg1=="BUY")
				{
				openalgo.PlaceOrder(tradetype_buyleg1,opttype_buyleg1,quantity_leg1,expiry_leg1,"buy","leg1");
				_TRACE("API Request leg1: "+api_request);
				_TRACE("API Response leg1: "+api_response);
				openalgo.PlaceOrder(tradetype_buyleg2,opttype_buyleg2,quantity_leg2,expiry_leg2,"buy","leg2");
				_TRACE("API Request leg2: "+api_request);
				_TRACE("API Response leg2: "+api_response);
				}
				else
				{
				openalgo.PlaceOrder(tradetype_buyleg2,opttype_buyleg2,quantity_leg2,expiry_leg2,"buy","leg2");
				_TRACE("API Request leg2: "+api_request);
				_TRACE("API Response leg2: "+api_response);
				openalgo.PlaceOrder(tradetype_buyleg1,opttype_buyleg1,quantity_leg1,expiry_leg1,"buy","leg1");
				_TRACE("API Request leg1: "+api_request);
				_TRACE("API Response leg1: "+api_response);
				}
				
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
            
				//Sell Signal
				if(tradetype_buyleg1=="SELL")
				{
				openalgo.ExitOrder(tradetype_buyleg1,opttype_buyleg1,expiry_leg1,"buy","leg1"); 
				_TRACE("Exit API Request leg1: "+ex_api_request);
				_TRACE("Exit API Response leg1: "+ex_api_response); 
				openalgo.ExitOrder(tradetype_buyleg2,opttype_buyleg2,expiry_leg2,"buy","leg2"); 
				_TRACE("Exit API Request leg2: "+ex_api_request);
				_TRACE("Exit API Response leg2: "+ex_api_response); 
				}
				else
				{
				openalgo.ExitOrder(tradetype_buyleg2,opttype_buyleg2,expiry_leg2,"buy","leg2");
				_TRACE("Exit API Request leg2: "+ex_api_request);
				_TRACE("Exit API Response leg2: "+ex_api_response); 
				openalgo.ExitOrder(tradetype_buyleg1,opttype_buyleg1,expiry_leg1,"buy","leg1");  
				_TRACE("Exit API Request leg2: "+ex_api_request);
				_TRACE("Exit API Response leg2: "+ex_api_response); 
				}
				
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
				//Short
				if(tradetype_shortleg1=="BUY")
				{
				openalgo.PlaceOrder(tradetype_shortleg1,opttype_shortleg1,quantity_leg1,expiry_leg1,"short","leg1");
				_TRACE("API Request leg1: "+api_request);
				_TRACE("API Response leg1: "+api_response);
				openalgo.PlaceOrder(tradetype_shortleg2,opttype_shortleg2,quantity_leg2,expiry_leg2,"short","leg2");
				_TRACE("API Request leg2: "+api_request);
				_TRACE("API Response leg2: "+api_response);
				}
				else
				{
				openalgo.PlaceOrder(tradetype_shortleg2,opttype_shortleg2,quantity_leg2,expiry_leg2,"short","leg2");
				_TRACE("API Request leg2: "+api_request);
				_TRACE("API Response leg2: "+api_response);
				openalgo.PlaceOrder(tradetype_shortleg1,opttype_shortleg1,quantity_leg1,expiry_leg1,"short","leg1");
				_TRACE("API Request leg1: "+api_request);
				_TRACE("API Response leg1: "+api_response);
				}
				
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
            
				//Cover Signal
				if(tradetype_shortleg1=="SELL")
				{
				openalgo.ExitOrder(tradetype_shortleg1,opttype_shortleg1,expiry_leg1,"short","leg1"); 
				_TRACE("Exit API Request leg1: "+ex_api_request);
				_TRACE("Exit API Response leg1: "+ex_api_response); 
				openalgo.ExitOrder(tradetype_shortleg2,opttype_shortleg2,expiry_leg2,"short","leg2"); 
				_TRACE("Exit API Request leg2: "+ex_api_request);
				_TRACE("Exit API Response leg2: "+ex_api_response); 
				}
				else
				{
				openalgo.ExitOrder(tradetype_shortleg2,opttype_shortleg2,expiry_leg2,"short","leg2");
				_TRACE("Exit API Request leg2: "+ex_api_request);
				_TRACE("Exit API Response leg2: "+ex_api_response); 
				openalgo.ExitOrder(tradetype_shortleg1,opttype_shortleg1,expiry_leg1,"short","leg1"); 
				_TRACE("Exit API Request leg1: "+ex_api_request);
				_TRACE("Exit API Response leg1: "+ex_api_response);  
				}
               
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

**Here are the Supported two-legged options trading strategies:**

**Credit Spread:** Selling an option at one strike price and buying an option at a lower strike price.

**Debit Spread:** Buying an option at one strike price and selling an option at a higher strike price.

**Straddle:** buying a call and a put option with the same strike price and expiration date.

**Strangle:** buying a call option with a higher strike price and a put option with a lower strike price

**Synthetic Futures:** Buying/Selling a call option and selling/Buying a put option of the same strike price and expiration date

**Diagonal Spread:** Buying a call or put option with a longer expiration date and selling a call or put option with a shorter expiration date and a different strike price.

**Calendar Spread:** Buying a call or put option with a longer expiration date and selling a call or put option with a shorter expiration date.

**Ratio Spread:** Buying a call or put option at one strike price and selling multiple options at a different strike price

**Ratio Back Spread:** Buying multiple options at one strike price and selling an option at a different strike price
