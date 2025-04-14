# Rate Limiting

Rate limiting defines limits on how many API calls can be made within a second.

The limit-exceeding requests will fail and return 403 Access denied because of exceeding rate limit.

| Sr. No | API Name        | Throttling Limit Rate (Request/Second) |
| ------ | --------------- | -------------------------------------- |
| 1      | PlaceOrder      | 10                                     |
| 2      | PlaceSmartOrder | 10                                     |
| 3      | ModifyOrder     | 10                                     |
| 4      | CancelOrder     | 10                                     |
| 5      | CancelAllOrder  | 10                                     |
| 6      | ClosePosition   | 10                                     |
