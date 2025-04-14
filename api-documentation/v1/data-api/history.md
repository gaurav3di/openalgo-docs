# History

## Endpoint URL

This API Function to fetch historical data from the Broker

```http
Local Host   :  POST http://127.0.0.1:5000/api/v1/history
Ngrok Domain :  POST https://<your-ngrok-domain>.ngrok-free.app/api/v1/history
Custom Domain:  POST https://<your-custom-domain>/api/v1/history
```



## Sample API Request

```json
{
    "apikey": "<your_app_apikey>",
    "symbol": "SBIN",
    "exchange": "NSE",
    "interval": "1m",
    "start_date": "2024-12-01",
    "end_date": "2024-12-31"
  
}

```

###

## Sample API Response

```json
{
  "data": [
    {
      "close": 836.4,
      "high": 841.1,
      "low": 836,
      "open": 838.95,
      "timestamp": 1733111100,
      "volume": 121671
    },
    {
      "close": 835.75,
      "high": 836.6,
      "low": 835.1,
      "open": 836.15,
      "timestamp": 1733111160,
      "volume": 40517
    },
    {
      "close": 835.2,
      "high": 836.15,
      "low": 834.5,
      "open": 835.75,
      "timestamp": 1733111220,
      "volume": 69600
    }
  ],
  "status": "success"
}
```



## Request Body



| Parameters  | Description                            | Mandatory/Optional | Default Value |
| ----------- | -------------------------------------- | ------------------ | ------------- |
| apikey      | App API key                            | Mandatory          | -             |
| symbol      | Trading symbol                         | Mandatory          | -             |
| exchange    | Exchange code                          | Mandatory          | -             |
| interval    | candle interval (see supported values) | Mandatory          | -             |
| start\_date | Start date (YYYY-MM-DD)                | Mandatory          | -             |
| end\_date   | End date (YYYY-MM-DD)                  | Mandatory          | -             |



## Response Fields

| Field     | Type   | Description          |
| --------- | ------ | -------------------- |
| timestamp | number | Unix epoch timestamp |
| open      | number | Opening price        |
| high      | number | High price           |
| low       | number | Low price            |
| close     | number | Closing price        |
| volume    | number | Trading volume       |



## Notes



1. Always check supported intervals first using the intervals API
2. Use exact interval strings from intervals API response
3. All timestamps are in Unix epoch format
