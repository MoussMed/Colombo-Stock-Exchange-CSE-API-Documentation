[![Releases](https://img.shields.io/badge/Releases-download-238636?style=flat-square)](https://github.com/MoussMed/Colombo-Stock-Exchange-CSE-API-Documentation/releases)

# Colombo Stock Exchange API Guide: CSE Python Examples

![Stock chart image](https://images.unsplash.com/photo-1555902716-6c6b5d2d4f7b?auto=format&fit=crop&w=1400&q=80)

Tags: colombo · cse · cse-api · exchange · stock · stock-market · stockdata

---

Table of contents
- About this repo
- Quick start
- Releases (download & run)
- API model (endpoints & payloads)
- Authentication
- Rate limits & best practices
- Python usage example
- Error handling and status codes
- Data mapping (fields & types)
- Pagination & filtering
- Realtime and WebSocket notes
- Testing and sandbox tips
- Contributing
- License
- Credits and resources

About this repo
This repository documents an unofficial API for the Colombo Stock Exchange (CSE). It describes endpoints, payloads, and common patterns. It includes a clear Python example that shows how to fetch instruments, historical prices, and live quotes. Use this as a usage guide and reference while you build clients, bots, or analytics workflows.

Quick start
- Clone or read this repo to get the endpoint list and examples.
- Install Python 3.8+ and requests.
- Visit the Releases page linked at the top for packaged examples and scripts.

Releases (download & run)
- The releases page hosts packaged scripts and example files. Download the release artifact that matches your OS and run the included script. Example: download cse-client-v1.0.zip, extract, then run python cse_client.py.
- Link: https://github.com/MoussMed/Colombo-Stock-Exchange-CSE-API-Documentation/releases
- If the releases page fails to load for any reason, check the "Releases" tab in this repository to find the packaged artifacts.

API model (endpoints & payloads)
This guide presents a typical set of endpoints you will find in CSE-like APIs. The endpoints below are examples and use JSON over HTTPS.

Base URL
- https://api.cse.lk/v1

Common endpoints
- GET /instruments
  - Returns meta for listed instruments (symbol, name, sector, lot size).
  - Query: ?search=, ?sector=, ?limit=50&page=1
- GET /instruments/{symbol}
  - Returns instrument detail and static metadata.
- GET /quotes/{symbol}
  - Returns last trade, bid/ask, volume, time.
- GET /historical/{symbol}
  - Returns OHLCV data. Query params: start=YYYY-MM-DD, end=YYYY-MM-DD, interval=daily|minute
- GET /market/summary
  - Returns market-level stats: indices, advancers, decliners, turnover.
- GET /corporate-actions/{symbol}
  - Returns dividends, splits, rights issues.
- POST /orders (if supported by broker integration)
  - Place orders via authenticated endpoint (requires secure credentials).

Request/response format
- Requests use JSON or query params for GETs.
- Responses use JSON. Dates use ISO 8601 in UTC.
- Numeric fields use basic types. Prices use float with two decimals, volumes use integers.

Authentication
- Typical methods: API key in header, OAuth2 bearer token, or HMAC for signed requests.
- Header example:
  - Authorization: Bearer <token>
  - X-API-KEY: <api_key>
- For order endpoints use short-lived tokens and TLS.

Rate limits & best practices
- Respect public rate limits: common pattern is 60 requests/minute. For heavier use, request a higher quota from the provider.
- Cache static resources (instruments, corporate actions).
- Use conditional GETs (ETag, If-Modified-Since) where supported.
- Back off on 429. Apply exponential backoff with jitter.

Python usage example
Install prerequisites
- pip install requests pandas

Example script (minimal)
```python
import requests
import pandas as pd
from datetime import datetime, timedelta

BASE = "https://api.cse.lk/v1"
HEADERS = {"Accept": "application/json", "X-API-KEY": "YOUR_API_KEY"}

def get_instruments(limit=200):
    url = f"{BASE}/instruments"
    params = {"limit": limit}
    r = requests.get(url, headers=HEADERS, params=params, timeout=10)
    r.raise_for_status()
    return r.json()

def get_historical(symbol, start, end, interval="daily"):
    url = f"{BASE}/historical/{symbol}"
    params = {"start": start, "end": end, "interval": interval}
    r = requests.get(url, headers=HEADERS, params=params, timeout=10)
    r.raise_for_status()
    return r.json()

if __name__ == "__main__":
    instruments = get_instruments(limit=100)
    print("Found instruments:", len(instruments.get("data", [])))

    # Example: fetch last 30 days for a symbol
    today = datetime.utcnow().date()
    start = (today - timedelta(days=30)).isoformat()
    end = today.isoformat()
    data = get_historical("CSE:ABC.N", start, end)
    df = pd.DataFrame(data.get("prices", []))
    print(df.head())
```

Notes on the example
- Replace YOUR_API_KEY with your token.
- Use CSV output or a database for persistent storage.
- Keep timezones consistent by storing UTC.

Error handling and status codes
- 200 OK: request succeeded.
- 201 Created: resource created (orders).
- 204 No Content: successful request with no body.
- 400 Bad Request: fix your request parameters.
- 401 Unauthorized: auth missing or invalid.
- 403 Forbidden: endpoint restricted.
- 404 Not Found: symbol or resource missing.
- 429 Too Many Requests: back off and retry later.
- 5xx Server Error: retry with backoff, report persistent issues.

Sample error payload
```json
{
  "error": {
    "code": 404,
    "message": "Instrument not found",
    "details": null
  }
}
```

Data mapping (fields & types)
Standard fields you will encounter:
- symbol (string): "CSE:ABC.N"
- name (string): "ABC PLC"
- last_price (float): 12.50
- open, high, low (float)
- close (float): closing price
- volume (int): traded shares
- turnover (float): value traded in local currency
- timestamp (string, ISO 8601): "2025-08-19T10:15:00Z"
- exchange (string): "CSE"

Pagination & filtering
- Use limit and page or cursor tokens.
- Example: GET /instruments?limit=100&page=2
- For historical data prefer date ranges to pages. Use server-side cursor when available.

Realtime and WebSocket notes
- Not all providers offer WebSocket.
- If available, use WebSocket for live quotes and trades. Use token-based auth and subscribe to channels: quotes.{symbol}, trades.{symbol}.
- Keep a heartbeat and reconnect on drop. Use sequence numbers to detect missed messages.
- Example subscribe message:
```json
{"action":"subscribe","channels":["quotes:CSE:ABC.N","trades:CSE:ABC.N"], "token":"YOUR_TOKEN"}
```

Testing and sandbox tips
- Use small requests and validate responses.
- Run the Python example against a sandbox if available.
- Mock endpoints with tools like httpbin or local Flask apps for integration tests.
- Store sample responses as fixtures for unit tests.

Security and privacy
- Keep API keys in environment variables or secret stores.
- Rotate keys regularly.
- Do not commit keys to source control.

Performance tuning
- Batch requests where possible.
- Use gzip compression and HTTP/2 if supported.
- Cache static lists and avoid polling high-frequency endpoints.

Common use cases
- Market data feed for dashboards
- Backtesting with historical OHLCV
- Alerting on price thresholds or volume spikes
- Portfolio tracking and valuation
- Automated trading with broker integrations (use strict risk controls)

Contributing
- Fork the repo and open a pull request.
- Add examples, correct endpoints, or update docs for new releases.
- Run tests and include sample responses.

Files in releases
- The releases page contains packaged scripts, one-line installers, and example CSV files.
- Download the artifact that matches your platform, extract, and execute the main script (for example: python cse_client.py).
- Use the Releases button above or go here: https://github.com/MoussMed/Colombo-Stock-Exchange-CSE-API-Documentation/releases

Examples of deliverables in releases
- cse-client-v1.0.zip — includes cse_client.py, requirements.txt, sample_config.json
- cse-data-tools-v1.0.tar.gz — CLI tools for download and convert
- cse-python-examples-v1.0.zip — ready-to-run scripts and notebooks

License
This documentation uses a permissive license. Check the LICENSE file in the repository for exact terms.

Credits and resources
- Colombo Stock Exchange official site (for public filings)
- Open-source libraries: requests, pandas, websocket-client
- Example images: Unsplash (stock trading photos)

Visual assets and badges
- Use shields to show releases, license, and topics.
- Example badges (replace URLs where needed):
  - [![Releases](https://img.shields.io/badge/Releases-download-238636?style=flat-square)](https://github.com/MoussMed/Colombo-Stock-Exchange-CSE-API-Documentation/releases)
  - [![Topics](https://img.shields.io/badge/topics-colombo%2C%20cse%2C%20stock%20market-blue?style=flat-square)](https://github.com/MoussMed/Colombo-Stock-Exchange-CSE-API-Documentation)

Contact and support
- Open an issue for corrections, missing endpoints, or suggested improvements.
- Send sample payloads or API responses to help expand this guide.

Images
- Use stock and finance images hosted on Unsplash or other free sources for readme art. Example:
  - https://images.unsplash.com/photo-1555902716-6c6b5d2d4f7b?auto=format&fit=crop&w=1400&q=80

This README documents a practical, hands-on approach to using a CSE-style API and includes a ready Python example. Use the Releases link above to download ready-made scripts and test files.