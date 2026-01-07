# MultiOptionGreeks

### **Multi Option Greeks API**

#### Endpoint URL <a href="#endpoint-url" id="endpoint-url"></a>

This API Function Calculates Multi Option Greeks (Delta, Gamma, Theta, Vega, Rho) and Implied Volatility using Black-76 Model

```
Local Host   :  POST http://127.0.0.1:5000/api/v1/multioptiongreeks
Ngrok Domain :  POST https://<your-ngrok-domain>.ngrok-free.app/api/v1/multioptiongreeks
Custom Domain:  POST https://<your-custom-domain>/api/v1/multioptiongreeks
```

**Why Black-76 Model?**

OpenAlgo uses the **Black-76 model** (via py\_vollib library) instead of Black-Scholes for calculating multi option Greeks. This is the correct choice for Indian F\&O markets:

**Key Benefits:**

* Accurate pricing for options on futures and forwards
* Industry-standard model used by NSE, BSE, MCX
* Greeks match with professional trading platforms
* Proper handling of cost of carry

#### Prerequisites <a href="#prerequisites" id="prerequisites"></a>

1. **py\_vollib Library Required**
   * Install with: `pip install py_vollib`
   * Or with uv: `uv pip install py_vollib`
   * Required for Black-76 calculations
2. **Market Data Access**
   * Requires real-time LTP for underlying and option
   * Uses OpenAlgo quotes API internally
3. **Valid API Key**
   * API key must be active and valid
   * Get API key from OpenAlgo settings

**Sample API Request**

```
{
  "apikey": "",
  "symbols": [
    {
      "symbol": "NIFTY27JAN2626000CE",
      "exchange": "NFO"
    },
    {
      "symbol": "NIFTY27JAN2626000PE",
      "exchange": "NFO"
    }
  ]
}
```

**Sample API Response (Success)**

```
{
  "data": [
    {
      "days_to_expiry": 20.2343,
      "exchange": "NFO",
      "expiry_date": "27-Jan-2026",
      "greeks": {
        "delta": 0.5633,
        "gamma": 0.000542,
        "rho": -0.191478,
        "theta": -7.0804,
        "vega": 24.2121
      },
      "implied_volatility": 11.83,
      "interest_rate": 0,
      "option_price": 345.4,
      "option_type": "CE",
      "spot_price": 26105.5,
      "status": "success",
      "strike": 26000,
      "symbol": "NIFTY27JAN2626000CE",
      "underlying": "NIFTY"
    },
    {
      "days_to_expiry": 20.2343,
      "exchange": "NFO",
      "expiry_date": "27-Jan-2026",
      "greeks": {
        "delta": -0.4006,
        "gamma": 0.000884,
        "rho": -0.069905,
        "theta": -4.1761,
        "vega": 23.7563
      },
      "implied_volatility": 7.11,
      "interest_rate": 0,
      "option_price": 126.1,
      "option_type": "PE",
      "spot_price": 26106.25,
      "status": "success",
      "strike": 26000,
      "symbol": "NIFTY27JAN2626000PE",
      "underlying": "NIFTY"
    }
  ],
  "status": "success",
  "summary": {
    "failed": 0,
    "success": 2,
    "total": 2
  }
}
```

### Parameter Description

| Parameters           | Description                                                                                                                                                                                          | Mandatory/Optional | Default Value                                          |
| -------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------ | ------------------------------------------------------ |
| apikey               | App API key                                                                                                                                                                                          | Mandatory          | -                                                      |
| symbol               | Option symbol (e.g., NIFTY02DEC2526000CE)                                                                                                                                                            | Mandatory          | -                                                      |
| exchange             | Exchange code (NFO, BFO, CDS, MCX)                                                                                                                                                                   | Mandatory          | -                                                      |
| interest\_rate       | Risk-free interest rate (annualized %). Specify current RBI repo rate (e.g., 6.5, 6.75) for accurate Rho calculations. Use 0 for theoretical calculations or when interest rate impact is negligible | Optional           | 0                                                      |
| forward\_price       | Custom forward/synthetic futures price. If provided, skips underlying price fetch. Useful for synthetic futures (Spot x e^rT) or illiquid underlyings like FINNIFTY, MIDCPNIFTY                      | Optional           | Auto-fetched                                           |
| underlying\_symbol   | Custom underlying symbol (e.g., NIFTY or NIFTY30DEC25FUT)                                                                                                                                            | Optional           | Auto-detected                                          |
| underlying\_exchange | Custom underlying exchange (e.g., NSE\_INDEX or NFO)                                                                                                                                                 | Optional           | Auto-detected                                          |
| expiry\_time         | Custom expiry time in HH:MM format (e.g., "17:00", "19:00"). Required for MCX contracts with non-standard expiry times                                                                               | Optional           | Exchange defaults: NFO/BFO=15:30, CDS=12:30, MCX=23:30 |

**Response Parameters**

| Parameter           | Description                             | Type   |
| ------------------- | --------------------------------------- | ------ |
| status              | API response status (success/error)     | string |
| symbol              | Option symbol                           | string |
| exchange            | Exchange code                           | string |
| underlying          | Underlying symbol                       | string |
| strike              | Strike price                            | number |
| option\_type        | Option type (CE/PE)                     | string |
| expiry\_date        | Expiry date (formatted)                 | string |
| days\_to\_expiry    | Days remaining to expiry                | number |
| spot\_price         | Underlying spot/futures/forward price   | number |
| option\_price       | Current option premium                  | number |
| interest\_rate      | Interest rate used                      | number |
| implied\_volatility | Implied Volatility (%)                  | number |
| greeks              | Object containing Greeks                | object |
| greeks.delta        | Delta (rate of change of option price)  | number |
| greeks.gamma        | Gamma (rate of change of delta)         | number |
| greeks.theta        | Theta (time decay per day)              | number |
| greeks.vega         | Vega (sensitivity to volatility per 1%) | number |
| greeks.rho          | Rho (sensitivity to interest rate)      | number |
