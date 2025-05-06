# Rate Limiting

To protect OpenAlgo from abuse and ensure fair usage across users, rate limits are enforced at both login and API levels. These limits are configurable via the `.env` file and apply globally per IP address or API key.

### UI Login Rate Limits

OpenAlgo applies two login-specific rate limits:

| Scope      | Limit        | Description                                      |
| ---------- | ------------ | ------------------------------------------------ |
| Per Minute | 5 per minute | Allows a maximum of 5 login attempts per minute. |
| Per Hour   | 25 per hour  | Allows a maximum of 25 login attempts per hour.  |

These limits help prevent brute-force login attempts and secure user accounts.

### API Rate Limit

General API calls are limited globally as follows:

| Scope      | Limit         | Description                                     |
| ---------- | ------------- | ----------------------------------------------- |
| Per Second | 10 per second | Allows a maximum of 10 API requests per second. |

This applies to all API endpoints other than login, including order placement, account info, and market data APIs.

### Configuration via .env

You can adjust the rate limits by editing the following variables in your `.env` or `.env.sample` file:

```
LOGIN_RATE_LIMIT_MIN="5 per minute"
LOGIN_RATE_LIMIT_HOUR="25 per hour"
API_RATE_LIMIT="10 per second"
```

These limits follow [Flask-Limiter syntax](https://flask-limiter.readthedocs.io/en/stable/#rate-limit-string-format) and support formats like:

* `10 per second`
* `100 per minute`
* `1000 per day`

### What Happens When Limits Are Exceeded

If a client exceeds any configured rate limit:

* The server will respond with HTTP status `429 Too Many Requests`.
* A `Retry-After` header will be sent with the time to wait before retrying.
* Further requests will be blocked until the rate window resets.

### Recommendations

* Avoid retrying failed login attempts rapidly.
* Spread out API requests using sleep/delay logic or a rate-limiter in your client code.
* Use queues or batching when dealing with large volumes of data or orders.

***
