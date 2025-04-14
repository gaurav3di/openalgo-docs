# OrderStatus

## Endpoint URL

```
Local Host   :  POST http://127.0.0.1:5000/api/v1/orderstatus
Ngrok Domain :  POST https://<your-ngrok-domain>.ngrok-free.app/api/v1/orderstatus
Custom Domain:  POST https://<your-custom-domain>/api/v1/orderstatus
```



## Sample API Request

```json
{
    "apikey":"<your_app_apikey>",
    "strategy": "Test Strategy",
    "orderid": "24120900146469"
}
```



## Sample API Response

```json
{
  "data": {
    "action": "BUY",
    "exchange": "NSE",
    "order_status": "complete",
    "orderid": "24120900146469",
    "price": 21.81,
    "pricetype": "MARKET",
    "product": "CNC",
    "quantity": 10,
    "symbol": "YESBANK",
    "timestamp": "09-Dec-2024 10:53:31",
    "trigger_price": 0
  },
  "status": "success"
}
```

###

## Parameters Description

| Parameters | Description   | Mandatory/Optional | Default Value |
| ---------- | ------------- | ------------------ | ------------- |
| apikey     | App API key   | Mandatory          | -             |
| strategy   | Strategy name | Mandatory          | -             |
| orderid    | Order Id      | Mandatory          | -             |
