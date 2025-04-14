# Depth

## Endpoint URL

This API Function get Market Depth from the Broker

```http
Local Host   :  POST http://127.0.0.1:5000/api/v1/depth
Ngrok Domain :  POST https://<your-ngrok-domain>.ngrok-free.app/api/v1/depth
Custom Domain:  POST https://<your-custom-domain>/api/v1/depth
```



## Sample API Request

```json
{
    "apikey": "<your_app_apikey>",
    "symbol": "RELIANCE",
    "exchange": "NSE"  
}

```

###

## Sample API Response

```json
{
  "data": {
    "asks": [
      {
        "price": 1311.55,
        "quantity": 5187
      },
      {
        "price": 0,
        "quantity": 0
      },
      {
        "price": 0,
        "quantity": 0
      },
      {
        "price": 0,
        "quantity": 0
      },
      {
        "price": 0,
        "quantity": 0
      }
    ],
    "bids": [
      {
        "price": 0,
        "quantity": 0
      },
      {
        "price": 0,
        "quantity": 0
      },
      {
        "price": 0,
        "quantity": 0
      },
      {
        "price": 0,
        "quantity": 0
      },
      {
        "price": 0,
        "quantity": 0
      }
    ],
    "high": 1323.9,
    "low": 1310,
    "ltp": 1311.55,
    "ltq": 10,
    "oi": 0,
    "open": 1323.9,
    "prev_close": 1311.55,
    "totalbuyqty": 0,
    "totalsellqty": 5187,
    "volume": 9037514
  },
  "status": "success"
}
```



## Request Body



| Parameters | Description    | Mandatory/Optional | Default Value |
| ---------- | -------------- | ------------------ | ------------- |
| apikey     | App API key    | Mandatory          | -             |
| symbol     | Trading symbol | Mandatory          | -             |
| exchange   | Exchange code  | Mandatory          | -             |



## Response Fields



| Field        | Type   | Description                  |
| ------------ | ------ | ---------------------------- |
| asks         | array  | List of 5 best ask prices    |
| bids         | array  | List of 5 best bid prices    |
| totalbuyqty  | number | Total buy quantity           |
| totalsellqty | number | Total sell quantity          |
| high         | number | Day's high price             |
| low          | number | Day's low price              |
| ltp          | number | Last traded price            |
| ltq          | number | Last traded quantity         |
| open         | number | Opening price                |
| prev\_close  | number | Previous day's closing price |
| volume       | number | Total traded volume          |
| oi           | number | Open interest                |
