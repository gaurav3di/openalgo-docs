# GttOrderBook

### **Endpoint URL**

This API Function lists **active** GTT triggers for the authenticated user. Triggered, cancelled, expired, and rejected GTTs are filtered out — only triggers that can still fire are returned.

```
Local Host   :  POST http://127.0.0.1:5000/api/v1/gttorderbook
Ngrok Domain :  POST https://<your-ngrok-domain>.ngrok-free.app/api/v1/gttorderbook
Custom Domain:  POST https://<your-custom-domain>/api/v1/gttorderbook
```

### **Sample API Request**

```json
{
    "apikey": "<your_app_apikey>"
}
```

### **Sample API Response**

```json
{
    "status": "success",
    "data": [
        {
            "trigger_id": "23132604291205",
            "trigger_type": "single",
            "status": "active",
            "symbol": "IDEA",
            "exchange": "NSE",
            "trigger_prices": [9.55],
            "last_price": 9.50,
            "legs": [
                {
                    "action": "BUY",
                    "quantity": 1,
                    "price": 9.50,
                    "pricetype": "LIMIT",
                    "product": "CNC"
                }
            ],
            "created_at": "2026-04-29 12:18:42",
            "updated_at": "",
            "expires_at": ""
        },
        {
            "trigger_id": "23132604291213",
            "trigger_type": "two-leg",
            "status": "active",
            "symbol": "INFY",
            "exchange": "NSE",
            "trigger_prices": [1480, 1620],
            "last_price": 1550,
            "legs": [
                {
                    "action": "SELL",
                    "quantity": 5,
                    "price": 1478,
                    "pricetype": "LIMIT",
                    "product": "CNC"
                },
                {
                    "action": "SELL",
                    "quantity": 5,
                    "price": 1622,
                    "pricetype": "LIMIT",
                    "product": "CNC"
                }
            ],
            "created_at": "2026-04-29 12:25:11",
            "updated_at": "",
            "expires_at": ""
        }
    ]
}
```

| Parameters | Description | Mandatory/Optional | Default Value |
| ---------- | ----------- | ------------------ | ------------- |
| apikey     | App API key | Mandatory          | -             |

**Response Fields**

| Field  | Description                 |
| ------ | --------------------------- |
| status | success or error            |
| data   | Array of active GTT entries |
