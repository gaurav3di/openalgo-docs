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

**Response Parameters**

{% columns %}
{% column %}
**Parameter**

status



symbol
{% endcolumn %}

{% column %}
**Description**

API response status (success/error)

Option symbol
{% endcolumn %}

{% column %}
**Type**

string



string
{% endcolumn %}
{% endcolumns %}

