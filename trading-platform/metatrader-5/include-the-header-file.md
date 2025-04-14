# Include the Header File

Sample code to Include the header file and creating input controls in your strategy

```cpp
#include <OpenAlgo/OpenAlgoApi.mqh>

input string ApiUrl = "http://127.0.0.1:5000";
input string ApiKey = "your_app_apikey";
input string Strategy = "Metatrader 5 Strategy";
input string Symbol = "SAIL-EQ";
input int Quantity = 1;

// Enums as input parameters
input Exchanges Exchange = NSE; // Default value set to NSE Equity
input ProductTypes Product = MIS; // Default value set to MIS for Intraday Square off
input PriceTypes PriceType = MARKET; // Default value set to Market Order

input int FastEMAPeriod = 5;
input int SlowEMAPeriod = 10;
```
