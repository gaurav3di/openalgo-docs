# Options Button Trading Module

This AFL script is an advanced interactive trading tool designed for AmiBroker, specifically to facilitate manual execution of options trades via a graphical interface connected to the OpenAlgo bridge API. It calculates key strike prices (ATM, ITM, OTM) for Call (CE) and Put (PE) options based on parameters such as the underlying spot, expiry, strike intervals, and offsets relative to the spot price (which can be based on previous open, close, or today’s open). It constructs standard option symbols for execution and visually displays the Algo status on the chart. The script also maintains static variables to track open trades and ensures strike recalculations align with user-selected inputs.

```clike
//Rajandran R - Creator of OpenAlgo
//website - openalgo.in / marketcalls.in
//OpenAlgo - Amibroker Options Button Trading Module v1.0
//Date - 20/08/2024



_SECTION_BEGIN("OpenAlgo Options Button Trading");

RequestTimedRefresh(1,False);

EnableTextOutput(False);

SetOption("StaticVarAutoSave", 30 );


apikey = ParamStr("OpenAlgo API Key", "******");

strategy = ParamStr("Strategy Name", "Test Strategy");

spot = Paramlist("Spot Symbol","NIFTY|BANKNIFTY|FINNIFTY|SENSEX|CRUDEOILM");  //OpenAlgo Underlying Symbol - used for Formulation of OpenAlgo Option Symbols
expiry = ParamStr("Expiry Date","17JUL25");

exchange = ParamList("Exchange","NFO|BFO|MCX",0); 
Symbol = ParamStr("Underlying Symbol(Data Vendor Symbol)","NIFTY"); // Amibroker Symbol - Data Vendors Symbol
iInterval= Param("Strike Interval",50,1,10000,1);
StrikeCalculation = Paramlist("Strike Calculation","PREVOPEN|PREVCLOSE|TODAYSOPEN",0);
LotSize = Param("Lot Size",75,1,10000,1);

ATMoffsetCE = Param("ATM CE Offset",0,-40,40,1);
ITMoffsetCE = Param("ITM CE Offset",-2,-40,-1,1);
OTMoffsetCE = Param("OTM CE Offset",4,1,40,1);

ATMoffsetPE = Param("ATM PE Offset",0,-40,40,1);
ITMoffsetPE = Param("ITM PE Offset",-2,-40,-1,1);
OTMoffsetPE = Param("OTM PE Offset",4,1,40,1);

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


EnableAlgo = ParamList("AlgoStatus","Disable|Enable",0);


bridgeurl = host+"/api/"+ver;
resp = "";


//Static Variables for Order protection

static_name_ = Name()+GetChartID()+interval(2)+strategy;
static_name_algo = Name()+GetChartID()+interval(2)+strategy+"algostatus";





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

if(StrikeCalculation=="PREVOPEN")
{
SetForeign(Symbol);
spotC = LastValue(Ref(OPEN,-1));
RestorePriceArrays();
}

if(StrikeCalculation=="PREVCLOSE")
{
SetForeign(Symbol);
spotC = LastValue(Ref(Close,-1));
RestorePriceArrays();
}

if(StrikeCalculation=="TODAYSOPEN")
{
SetForeign(Symbol);
spotC = LastValue(TimeFrameGetPrice("O",inDaily));
RestorePriceArrays();
}

//Maintain Array to Store ATM Strikes for each and every bar
strike = IIf(spotC % iInterval > iInterval/2, spotC - (spotC%iInterval) + iInterval,
			spotC - (spotC%iInterval));
			
//ATM/ITM/OTM Entry/Exit Strikes	
		
ATMstrikeCE = strike + (ATMoffsetCE * iInterval);
ATMstrikePE = strike - (ATMoffsetPE * iInterval);

ITMstrikeCE = strike + (ITMoffsetCE * iInterval);
ITMstrikePE = strike - (ITMoffsetPE * iInterval);

OTMstrikeCE = strike + (OTMoffsetCE * iInterval);
OTMstrikePE = strike - (OTMoffsetPE * iInterval);

//ATM/ITM/OTM OpenAlgoSymbol Format

ATMsymbolCE = spot+expiry+ATMstrikeCE+"CE";
ATMsymbolPE = spot+expiry+ATMstrikePE+"PE";

ITMsymbolCE = spot+expiry+ITMstrikeCE+"CE";
ITMsymbolPE = spot+expiry+ITMstrikePE+"PE";

OTMsymbolCE = spot+expiry+OTMstrikeCE+"CE";
OTMsymbolPE = spot+expiry+OTMstrikePE+"PE";


printf("\nATMsymbolCE = "+ATMsymbolCE);
printf("\nATMsymbolPE = "+ATMsymbolPE);

printf("\nITMsymbolCE = "+ITMsymbolCE);
printf("\nITMsymbolPE = "+ITMsymbolPE);

printf("\nOTMsymbolCE = "+OTMsymbolCE);
printf("\nOTMsymbolPE = "+OTMsymbolPE);

printf("\n ------Internal Memory-------");
printf("\n ATMsymbolCE is : " + StaticVarGetText(static_name_+"ATMsymbolCE"));
printf("\n ITMsymbolCE is : " + StaticVarGetText(static_name_+"ITMsymbolCE"));
printf("\n OTMsymbolCE is : " + StaticVarGetText(static_name_+"OTMsymbolCE"));

printf("\n ATMsymbolPE is : " + StaticVarGetText(static_name_+"ATMsymbolPE"));
printf("\n ITMsymbolPE is : " + StaticVarGetText(static_name_+"ITMsymbolPE"));
printf("\n OTMsymbolPE is : " + StaticVarGetText(static_name_+"OTMsymbolPE"));




_SECTION_BEGIN("OpenAlgo Bridge Controls");

EnableScript("VBScript"); 
<%
Public Sub PlaceOrder(symbol,action, quantity)
	
    Dim oXMLHTTP
    Dim oStream
    Set oXMLHTTP = CreateObject("Msxml2.XMLHTTP")
    ' Define variables with the specified values
    Dim apikey, strategy , exchange, pricetype, product
    apikey = AFL.Var("apikey")
    strategy = AFL.Var("strategy")
    
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


Public Sub ExitOrder(symbol)
    Dim oXMLHTTP
    Dim oStream
    Set oXMLHTTP = CreateObject("Msxml2.XMLHTTP")
    ' Define variables with the specified values
    Dim apikey, strategy , exchange, pricetype, product
    apikey = AFL.Var("apikey")
    strategy = AFL.Var("strategy")
    
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
    """,""action"":""" & "SELL" & _
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



Public Sub SquareoffAll()
    Dim oXMLHTTP
    Dim oStream
    Set oXMLHTTP = CreateObject("Msxml2.XMLHTTP")
    ' Define variables with the specified values
    Dim apikey, strategy
    apikey = AFL.Var("apikey")
    strategy = AFL.Var("strategy")
      
    
    ' Construct the JSON string for the POST message
    Dim jsonRequestBody
    jsonRequestBody = "{""apikey"":""" & apikey & _
    """,""strategy"":""" & strategy & """}"
    
    ' Set the URL
    Dim url
    url = AFL.Var("bridgeurl")&"/closeposition"
    
    ' Configure the HTTP request for POST method
    oXMLHTTP.Open "POST", url, False
    oXMLHTTP.setRequestHeader "Content-Type", "application/json"
    oXMLHTTP.setRequestHeader "Cache-Control", "no-cache"
    oXMLHTTP.setRequestHeader "Pragma", "no-cache"
    
    ' Send the request with the JSON body
    oXMLHTTP.Send jsonRequestBody
    
    api_parameters = "Strategy :" & strategy & " api_url :" & url

    
    AFL("sq_api_request") = api_parameters  
    AFL("sq_api_response") = oXMLHTTP.responseText
    
    
    ' Optionally, handle the response here
    ' Dim response
    ' response = oXMLHTTP.responseText
    ' Response handling code...
End Sub


%>


openalgo = GetScriptObject();




_SECTION_BEGIN("Button Trading -  For Old Amibroker Versions");

X0 = 20;
Y0 = 100;
X1 = 60;

X2 = 20;
Y2 = 240;
X3 = 60;

LBClick = GetCursorMouseButtons() == 9;	// Click
MouseX  = Nz(GetCursorXPosition(1));		// 
MouseY  = Nz(GetCursorYPosition(1));		//

procedure DrawButton (Text, x1, y1, x2, y2, colorFrom, colorTo)
{
	GfxSetOverlayMode(0);
	GfxSelectFont("Verdana", 9, 700);
	GfxSetBkMode(1);
	GfxGradientRect(x1, y1, x2, y2, colorFrom, colorTo);
	GfxDrawText(Text, x1, y1, x2, y2, 32|1|4|16);
}
GfxSetTextColor(colorWhite);

if(EnableAlgo == "Enable")
{  

	DrawButton("ATM CE", X0, Y0, X0+X1+50, Y0+50, colorGreen, colorGreen);
	CursorInATMCEButton = MouseX >= X0 AND MouseX <= X0+X1+50 AND MouseY >= Y0 AND MouseY <= Y0+50;
	ATMCEButtonClick = CursorInATMCEButton AND LBClick;
	
	DrawButton("ITM CE", X0+115, Y0, X0+X1+165, Y0+50, colorRed, colorRed);
	CursorInITMCEButton = MouseX >= X0+115 AND MouseX <= X0+X1+165 AND MouseY >= Y0 AND MouseY <= Y0+50;
	ITMCEButtonClick = CursorInITMCEButton AND LBClick;
	
	DrawButton("OTM CE", X0+230, Y0, X0+X1+280, Y0+50, colorBlue, colorBlue);
	CursorInOTMCEButton = MouseX >= X0+230 AND MouseX <= X0+X1+280 AND MouseY >= Y0 AND MouseY <= Y0+50;
	OTMCEButtonClick = CursorInOTMCEButton AND LBClick;
	
	DrawButton("x ATM CE", X0, Y0+55, X0+X1+50, Y0+105, colorGreen, colorGreen);
	xCursorInATMCEButton = MouseX >= X0 AND MouseX <= X0+X1+50 AND MouseY >= Y0+55 AND MouseY <= Y0+105;
	xATMCEButtonClick = xCursorInATMCEButton AND LBClick;
	
	DrawButton("x ITM CE", X0+115, Y0+55, X0+X1+165, Y0+105, colorRed, colorRed);
	xCursorInITMCEButton = MouseX >= X0+115 AND MouseX <= X0+X1+165 AND MouseY >= Y0+55 AND MouseY <= Y0+105;
	xITMCEButtonClick = xCursorInITMCEButton AND LBClick;
	
	DrawButton("x OTM CE", X0+230, Y0+55, X0+X1+280, Y0+105, colorBlue, colorBlue);
	xCursorInOTMCEButton = MouseX >= X0+230 AND MouseX <= X0+X1+280 AND MouseY >= Y0+55 AND MouseY <= Y0+105;
	xOTMCEButtonClick = xCursorInOTMCEButton AND LBClick;
	
	
	DrawButton("ATM PE", X2, Y2, X2+X3+50, Y2+50, colorGreen, colorGreen);
	CursorInATMPEButton = MouseX >= X2 AND MouseX <= X2+X3+50 AND MouseY >= Y2 AND MouseY <= Y2+50;
	ATMPEButtonClick = CursorInATMPEButton AND LBClick;
	
	DrawButton("ITM PE", X2+115, Y2, X2+X3+165, Y2+50, colorRed, colorRed);
	CursorInITMPEButton = MouseX >= X2+115 AND MouseX <= X2+X3+165 AND MouseY >= Y2 AND MouseY <= Y2+50;
	ITMPEButtonClick = CursorInITMPEButton AND LBClick;
	
	DrawButton("OTM PE", X2+230, Y2, X2+X3+280, Y2+50, colorBlue, colorBlue);
	CursorInOTMPEButton = MouseX >= X2+230 AND MouseX <= X2+X3+280 AND MouseY >= Y2 AND MouseY <= Y2+50;
	OTMPEButtonClick = CursorInOTMPEButton AND LBClick;
	
	DrawButton("x ATM PE", X2, Y2+55, X2+X3+50, Y2+105, colorGreen, colorGreen);
	xCursorInATMPEButton = MouseX >= X2 AND MouseX <= X2+X3+50 AND MouseY >= Y2+55 AND MouseY <= Y2+105;
	xATMPEButtonClick = xCursorInATMPEButton AND LBClick;
	
	DrawButton("x ITM PE", X2+115, Y2+55, X2+X3+165, Y2+105, colorRed, colorRed);
	xCursorInITMPEButton = MouseX >= X2+115 AND MouseX <= X2+X3+165 AND MouseY >= Y2+55 AND MouseY <= Y2+105;
	xITMPEButtonClick = xCursorInITMPEButton AND LBClick;
	
	DrawButton("x OTM PE", X2+230, Y2+55, X2+X3+280, Y2+105, colorBlue, colorBlue);
	xCursorInOTMPEButton = MouseX >= X2+230 AND MouseX <= X2+X3+280 AND MouseY >= Y2+55 AND MouseY <= Y2+105;
	xOTMPEButtonClick = xCursorInOTMPEButton AND LBClick;
	
	
	DrawButton("CLOSE ALL", X0, Y0+270, X0+X1+65, Y0+320, colorRed, colorRed);
	CursorInCXButton = MouseX >= X0 AND MouseX <= X0+X1+65 AND MouseY >= Y0+270 AND MouseY <= Y0+320;
	CXButtonClick = CursorInCXButton AND LBClick;
	
	
	
	// Logic for all buttons
	// ATM CE entry and exit logic
	if (ATMCEButtonClick AND StaticVarGet(static_name_ + "ATMCEAlgo") == 0) {
		_TRACE("Placing ATM CE Entry Order");
		openalgo.placeorder(ATMsymbolCE, tradetype, quantity);
		StaticVarSetText(static_name_+"ATMsymbolCE",ATMsymbolCE,True);
		if (VoiceAlert == "Enable") { Say("ATM CE Order Triggered"); }
		_TRACE("API Request : " + api_request);
		_TRACE("API Response : " + api_response);
		StaticVarSet(static_name_ + "ATMCEAlgo", 1);
	} else { StaticVarSet(static_name_ + "ATMCEAlgo", 0); }

	if (xATMCEButtonClick AND StaticVarGet(static_name_ + "xATMCEAlgo") == 0) {
		_TRACE("Placing ATM CE Exit Order");
		exitsymbol = StaticVarGetText(static_name_+"ATMsymbolCE");
		if(exitsymbol!="")
		{
		openalgo.exitorder(exitsymbol);
		StaticVarSetText(static_name_+"ATMsymbolCE","");
		if (VoiceAlert == "Enable") { Say("Exit ATM CE Order Triggered"); }
		_TRACE("API Request : " + ex_api_request);
		_TRACE("API Response : " + ex_api_response);
		StaticVarSet(static_name_ + "xATMCEAlgo", 1);
		}
		else
		{
		Say("No Open Orders to Exit");
		_TRACE("No Open Orders to Exit");
		}
	} else { StaticVarSet(static_name_ + "xATMCEAlgo", 0); }

	// ITM CE entry and exit logic
	if (ITMCEButtonClick AND StaticVarGet(static_name_ + "ITMCEAlgo") == 0) {
		_TRACE("Placing ITM CE Entry Order");
		openalgo.placeorder(ITMsymbolCE, tradetype, quantity);
		StaticVarSetText(static_name_+"ITMsymbolCE",ITMsymbolCE,True);
		if (VoiceAlert == "Enable") { Say("ITM CE Order Triggered"); }
		_TRACE("API Request : " + api_request);
		_TRACE("API Response : " + api_response);
		StaticVarSet(static_name_ + "ITMCEAlgo", 1);
	} else { StaticVarSet(static_name_ + "ITMCEAlgo", 0); }

	if (xITMCEButtonClick AND StaticVarGet(static_name_ + "xITMCEAlgo") == 0) {
		_TRACE("Placing ITM CE Exit Order");
		exitsymbol = StaticVarGetText(static_name_+"ITMsymbolCE");
		if(exitsymbol!="")
		{
		openalgo.exitorder(exitsymbol);
		StaticVarSetText(static_name_+"ITMsymbolCE","");
		if (VoiceAlert == "Enable") { Say("Exit ITM CE Order Triggered"); }
		_TRACE("API Request : " + ex_api_request);
		_TRACE("API Response : " + ex_api_response);
		StaticVarSet(static_name_ + "xITMCEAlgo", 1);
		}
		else
		{
		Say("No Open Orders to Exit");
		_TRACE("No Open Orders to Exit");
		}
	} else { StaticVarSet(static_name_ + "xITMCEAlgo", 0); }

	// OTM CE entry and exit logic
	if (OTMCEButtonClick AND StaticVarGet(static_name_ + "OTMCEAlgo") == 0) {
		_TRACE("Placing OTM CE Entry Order");
		openalgo.placeorder(OTMsymbolCE, tradetype, quantity);
		StaticVarSetText(static_name_+"OTMsymbolCE",OTMsymbolCE,True);
		if (VoiceAlert == "Enable") { Say("OTM CE Order Triggered"); }
		_TRACE("API Request : " + api_request);
		_TRACE("API Response : " + api_response);
		StaticVarSet(static_name_ + "OTMCEAlgo", 1);
	} else { StaticVarSet(static_name_ + "OTMCEAlgo", 0); }

	if (xOTMCEButtonClick AND StaticVarGet(static_name_ + "xOTMCEAlgo") == 0) {
		_TRACE("Placing OTM CE Exit Order");
		exitsymbol = StaticVarGetText(static_name_+"OTMsymbolCE");
		if(exitsymbol!="")
		{
		openalgo.exitorder(exitsymbol);
		StaticVarSetText(static_name_+"OTMsymbolCE","");
		if (VoiceAlert == "Enable") { Say("Exit OTM CE Order Triggered"); }
		_TRACE("API Request : " + ex_api_request);
		_TRACE("API Response : " + ex_api_response);
		StaticVarSet(static_name_ + "xOTMCEAlgo", 1);
		}
		else
		{
		Say("No Open Orders to Exit");
		_TRACE("No Open Orders to Exit");
		}
	} else { StaticVarSet(static_name_ + "xOTMCEAlgo", 0); }

	// ATM PE entry and exit logic
	if (ATMPEButtonClick AND StaticVarGet(static_name_ + "ATMPEAlgo") == 0) {
		_TRACE("Placing ATM PE Entry Order");
		openalgo.placeorder(ATMsymbolPE, tradetype, quantity);
		StaticVarSetText(static_name_+"ATMsymbolPE",ATMsymbolPE,True);
		if (VoiceAlert == "Enable") { Say("ATM PE Order Triggered"); }
		_TRACE("API Request : " + api_request);
		_TRACE("API Response : " + api_response);
		StaticVarSet(static_name_ + "ATMPEAlgo", 1);
	} else { StaticVarSet(static_name_ + "ATMPEAlgo", 0); }

	if (xATMPEButtonClick AND StaticVarGet(static_name_ + "xATMPEAlgo") == 0) {
		_TRACE("Placing ATM PE Exit Order");
		exitsymbol = StaticVarGetText(static_name_+"ATMsymbolPE");
		if(exitsymbol!="")
		{
		openalgo.exitorder(exitsymbol);
		StaticVarSetText(static_name_+"ATMsymbolPE","");
		if (VoiceAlert == "Enable") { Say("Exit ATM PE Order Triggered"); }
		_TRACE("API Request : " + ex_api_request);
		_TRACE("API Response : " + ex_api_response);
		StaticVarSet(static_name_ + "xATMPEAlgo", 1);
		}
		else
		{
		Say("No Open Orders to Exit");
		_TRACE("No Open Orders to Exit");
		}
	} else { StaticVarSet(static_name_ + "xATMPEAlgo", 0); }

	// ITM PE entry and exit logic
	if (ITMPEButtonClick AND StaticVarGet(static_name_ + "ITMPEAlgo") == 0) {
		_TRACE("Placing ITM PE Entry Order");
		openalgo.placeorder(ITMsymbolPE, tradetype, quantity);
		StaticVarSetText(static_name_+"ITMsymbolPE",ITMsymbolPE,True);
		if (VoiceAlert == "Enable") { Say("ITM PE Order Triggered"); }
		_TRACE("API Request : " + api_request);
		_TRACE("API Response : " + api_response);
		StaticVarSet(static_name_ + "ITMPEAlgo", 1);

	} else { StaticVarSet(static_name_ + "ITMPEAlgo", 0); }

	if (xITMPEButtonClick AND StaticVarGet(static_name_ + "xITMPEAlgo") == 0) {
		_TRACE("Placing ITM PE Exit Order");
				exitsymbol = StaticVarGetText(static_name_+"ITMsymbolPE");
		if(exitsymbol!="")
		{
		openalgo.exitorder(exitsymbol);
		StaticVarSetText(static_name_+"ITMsymbolPE","");
		if (VoiceAlert == "Enable") { Say("Exit ITM PE Order Triggered"); }
		_TRACE("API Request : " + ex_api_request);
		_TRACE("API Response : " + ex_api_response);
		StaticVarSet(static_name_ + "xITMPEAlgo", 1);
		}
		else
		{
		Say("No Open Orders to Exit");
		_TRACE("No Open Orders to Exit");
		}
	} else { StaticVarSet(static_name_ + "xITMPEAlgo", 0); }

	// OTM PE entry and exit logic
	if (OTMPEButtonClick AND StaticVarGet(static_name_ + "OTMPEAlgo") == 0) {
		_TRACE("Placing OTM PE Entry Order");
		openalgo.placeorder(OTMsymbolPE, tradetype, quantity);
		StaticVarSetText(static_name_+"OTMsymbolPE",OTMsymbolPE,True);
		if (VoiceAlert == "Enable") { Say("OTM PE Order Triggered"); }
		_TRACE("API Request : " + api_request);
		_TRACE("API Response : " + api_response);
		StaticVarSet(static_name_ + "OTMPEAlgo", 1);
	} else { StaticVarSet(static_name_ + "OTMPEAlgo", 0); }

	if (xOTMPEButtonClick AND StaticVarGet(static_name_ + "xOTMPEAlgo") == 0) {
		_TRACE("Placing OTM PE Exit Order");
		exitsymbol = StaticVarGetText(static_name_+"OTMsymbolPE");
		if(exitsymbol!="")
		{
		openalgo.exitorder(exitsymbol);
		StaticVarSetText(static_name_+"OTMsymbolPE","");
		if (VoiceAlert == "Enable") { Say("Exit OTM PE Order Triggered"); }
		_TRACE("API Request : " + ex_api_request);
		_TRACE("API Response : " + ex_api_response);
		StaticVarSet(static_name_ + "xOTMPEAlgo", 1);
		}
		else
		{
		Say("No Open Orders to Exit");
		_TRACE("No Open Orders to Exit");
		}
	} else { StaticVarSet(static_name_ + "xOTMPEAlgo", 0); }


	
	
	if( CXButtonClick AND StaticVarGet(Name()+GetChartID()+"CXAlgo")==0 ) 
	{
		openalgo.Squareoffall();
		if(VoiceAlert == "Enable"){
				Say("Squareoff All Triggered");  	
			}
		_TRACE("API Request : "+sq_api_request);
		_TRACE("API Response : "+sq_api_response);
		StaticVarSet(Name()+GetChartID()+"CXAlgo",1); 
	}
	else
	{
		StaticVarSet(Name()+GetChartID()+"CXAlgo",0);
	}
	
	
}

_SECTION_END();


_SECTION_BEGIN("Candlestick Charts with Date & Time Axis");

//Enable the Date & Time Axis
SetChartOptions(0, chartShowArrows | chartShowDates);

//Plotting Candlestick charts
Plot(Close,"Candle",colorDefault,styleCandle);


_SECTION_END();

```
