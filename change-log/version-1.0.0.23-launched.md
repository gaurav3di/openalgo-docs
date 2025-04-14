# Version 1.0.0.23 Launched

12th Apr 2025

**New Broker Additions (XTS API + Direct API):**\
• 5paisa (XTS)\
• Compositedge (XTS)\
• IIFL (XTS)\
• Jainam (Retail & Dealer - XTS)\
• Wisdom Capital (XTS)\
• Paytm

🔧 **Broker-Specific Fixes & Enhancements:**\
• **AngelOne:**\
 – Tick Size Handling Fix\
 – Index Symbol Normalization\
 – Historical Data Looping Fix via API\
 – Margin Format Standardization\
 – ExportCSV for LTP and PnL\
 – Migrated to `httpx` with Connection Pooling

• **XTS API Supported Brokers:**\
 – Centralized Base URL, Added Feedtoken and UserID to the Database

• **Flattrade:**\
 – Quote Formatting\
 – Common Symbol Mapping

• **Fyers:**\
 – BFO Master Contract Download Fix\
 – Implemented Common Symbol Format for NSE/BSE

• **Pocketful:**\
 – Initial Integration: Direct Auth & Fund Fetch

• **Zerodha, Dhan, Shoonya, Upstox, Kotak:**\
 – Symbol Format & Index Normalization\
 – Historical Quote Adjustments

**Common Symbol Format Now Implemented For:**

1. Fyers
2. Dhan
3. Zerodha
4. Flattrade
5. Shoonya _(BSE indices not available)_
6. Compositedge
7. AngelOne
8. Upstox
9. Kotak _(BSE indices not streaming)_
10. Paytm
11. Jainam (XTS API)
12. IIFL (XTS API)
13. Wisdom Capital (XTS API)
14. 5paisa (XTS API)



⚙️ **System Architecture & Enhancements:**\
• Connection Pooling Enabled (AngelOne, PayTM and  XTS Supported Brokers)\
• `user_id` Column Added \
• Python 3.13 Compatibility Achieved (Updated Dependencies)

📄 **Documentation & DevOps Updates:**\
• Refined Docs: README, Design Overview, XTS/XTSAPI Guides\
• Updated: `.env` Samples, Broker Auto-Config, Shell Installer\
