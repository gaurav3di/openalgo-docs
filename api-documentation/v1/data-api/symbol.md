# Symbol

## Endpoint URL

This API Function fetches Quotes from the Broker

```http
Local Host   :  POST http://127.0.0.1:5000/api/v1/symbol
Ngrok Domain :  POST https://<your-ngrok-domain>.ngrok-free.app/api/v1/symbol
Custom Domain:  POST https://<your-custom-domain>/api/v1/symbol
```



## Sample API Request

```json
{
    "apikey": "<your_app_apikey>",
    "symbol":"SBIN", 
    "exchange":"NSE"     
}

```

###

## Sample API Response

```json
{
  "data": {
    "brexchange": "NSE",
    "brsymbol": "ZOMATO-EQ",
    "exchange": "NSE",
    "expiry": "",
    "id": 7162,
    "instrumenttype": "",
    "lotsize": 1,
    "name": "ZOMATO",
    "strike": -0.01,
    "symbol": "ZOMATO",
    "tick_size": 0.01,
    "token": "5097"
  },
  "status": "success"
}
```



## Request Fields



| Parameters | Description    | Mandatory/Optional | Default Value |
| ---------- | -------------- | ------------------ | ------------- |
| apikey     | App API key    | Mandatory          | -             |
| symbol     | Trading symbol | Mandatory          | -             |
| exchange   | Exchange code  | Mandatory          | -             |





## Response Fields

| Field          | Type    | Description                                |
| -------------- | ------- | ------------------------------------------ |
| brexchange     | String  | Broker exchange name                       |
| brsymbol       | String  | Broker-specific symbol                     |
| exchange       | String  | Exchange name (e.g., NSE, BSE)             |
| expiry         | String  | Expiry date (empty for equity instruments) |
| id             | Integer | Unique instrument identifier               |
| instrumenttype | String  | Type of instrument (e.g., EQ, FUT, OPT)    |
| lotsize        | Integer | Lot size (number of units per lot)         |
| name           | String  | Name of the instrument/company             |
| strike         | Float   | Strike price (used in options, -0.01 here) |
| symbol         | String  | Trading symbol                             |
| tick\_size     | Float   | Minimum price movement                     |
| token          | String  | Token ID                                   |
