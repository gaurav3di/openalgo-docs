# Quotes

## Endpoint URL

This API Function fetches Quotes from the Broker

```http
Local Host   :  POST http://127.0.0.1:5000/api/v1/quotes
Ngrok Domain :  POST https://<your-ngrok-domain>.ngrok-free.app/api/v1/quotes
Custom Domain:  POST https://<your-custom-domain>/api/v1/quotes
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
    "ask": 863.65,
    "bid": 0,
    "high": 875.45,
    "low": 857,
    "ltp": 863.65,
    "open": 865.25,
    "prev_close": 863.65,
    "volume": 14520568
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



| Field       | Type   | Description                  |
| ----------- | ------ | ---------------------------- |
| bid         | number | Best bid price               |
| ask         | number | Best ask price               |
| open        | number | Opening price                |
| high        | number | High price                   |
| low         | number | Low price                    |
| ltp         | number | Last traded price            |
| prev\_close | number | Previous day's closing price |
| volume      | number | Total traded volume          |
