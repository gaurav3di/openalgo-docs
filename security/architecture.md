# Architecture

### OpenAlgo Security Architecture

OpenAlgo implements multiple layers of security to protect your trading operations, data, and infrastructure.

***

1. **Authentication & Session Management**

* Encrypted Password Storage: User passwords are hashed using Argon2 (industry-standard, resistant to GPU attacks)
* Session Expiry: Automatic daily session expiration at configurable time (default 3:30 AM IST)
* Secure Session Cookies: HTTPOnly, SameSite, and Secure flags enabled for HTTPS deployments
* Token Revocation: Automatic token revocation on session expiry or logout

***

2. **API Security**

* API Key Authentication: Unique API keys for each user with Argon2 hashing + pepper
* Encrypted Token Storage: All broker tokens encrypted using Fernet symmetric encryption
* API Rate Limiting: Configurable rate limits on all API endpoints (default 10-60 requests/minute)
* Invalid API Key Tracking: Auto-bans IPs after 10 invalid API key attempts

***

3. **Network Security**

* IP Ban System: Automatic and manual IP blocking for malicious traffic
* 404 Attack Detection: Auto-bans after 20 suspicious 404 errors per day
* CSRF Protection: Built-in CSRF tokens on all forms and state-changing operations
* CORS Configuration: Restrictive cross-origin policies with configurable allowed origins
* Content Security Policy (CSP): Prevents XSS attacks through strict CSP headers

***

4. **Data Protection**

* Database Encryption: Sensitive data (API keys, broker tokens, SMTP passwords) encrypted at rest
* No Credential Logging: API keys and passwords never logged in plaintext
* Secure Configuration: Environment variables for sensitive configuration
* Pepper-based Hashing: Additional pepper secret for password and API key hashing

***

5. **Infrastructure Security**

* Security Middleware: Request validation and IP checking before processing
* Traffic Logging: Complete audit trail of all requests with anonymized sensitive data
* Performance Monitoring: Latency tracking to detect DoS attempts
* Error Handling: Generic error messages to prevent information leakage
* File Upload Restrictions: Limited file types and size restrictions

***

6. **Broker Integration Security**

* OAuth 2.0 Support: Secure authorization flow for broker connections
* Token Refresh: Automatic token refresh without storing credentials
* Broker-specific Encryption: Each broker's tokens encrypted separately
* Revocation Support: Ability to immediately revoke broker access

***

7. **WebSocket Security**

* Authentication Required: WebSocket connections require valid session
* Message Validation: All WebSocket messages validated before processing
* Connection Limits: Maximum concurrent connections per user
* Heartbeat Monitoring: Automatic disconnection of idle connections

***

8. **Operational Security**

* Security Dashboard: Real-time monitoring at /security
* Audit Logs: Comprehensive logging of all security events
* Configurable Thresholds: Adjust security settings without code changes
* Multiple Ban Durations: Temporary (24hr, 48hr, 1 week) or permanent bans
* Repeat Offender Detection: Escalating penalties for repeat violations

***

9. **Development Security**

* No Hardcoded Secrets: All sensitive data in environment variables or database
* Input Validation: All user inputs sanitized and validated
* SQL Injection Prevention: Using SQLAlchemy ORM with parameterized queries
* XSS Prevention: Template auto-escaping and CSP headers
* Dependency Management: Regular updates and security patches

***

10. **Deployment Security**

* HTTPS Support: Automatic secure cookie configuration for HTTPS
* Docker Security: Non-root user in container, minimal base images

***

**Security Best Practices for Users**

1. Use Strong API Keys: Generate complex, random API keys
2. Enable HTTPS: Always use SSL certificates in production
3. Regular Updates: Keep OpenAlgo updated for latest security patches
4. Monitor Security Dashboard: Regular checks at /security
5. Backup Before Bans: Export data before applying permanent bans
6. Whitelist Trusted IPs: (Coming soon) Add trusted IPs to whitelist
7. Review Logs: Regularly check /logs for suspicious activity
8. Secure Environment: Protect your .env file and database
