# Matt: IT Inventory System (case study)

Internal tool I built to control IT equipment inventory (monitors, headsets, mice, printers) with real-time sync to Google Sheets. The source code is private. This repo only describes the design and shows simplified samples.

## What it does

**Inventory and transactions:** equipment stock, entries and exits, suppliers, budget and loss dashboard.

**Invoice intake:** NF-e entry from XML, image or SEFAZ key.

**AI assistant:** a chatbot (Vertex AI) that answers questions about the stock.

**Printers:** status and ink monitoring.

**Multi-user and multi-language:** everyone sees the same data, with PT, EN and ES interface.

## Stack

Python, Flask (REST API), SQLite, Google Sheets API, Vertex AI, HTML/JS frontend, Docker, HTTPS reverse proxy, optional S3 backup of the database.

## Architecture

Browser -> Flask API -> in-memory cache -> Google Sheets (source of truth shared by the team) with SQLite as local store. A global update timestamp lets every client auto-refresh when someone changes data.

## Design decisions

**Rate limiting for the Sheets API:** Google limits requests per minute, so every call goes through a small limiter (conservative maximum per minute and a minimum interval between calls). This avoids quota errors when many users are active.

**Cache that never serves an empty list:** when Sheets fails temporarily, the API returns the last valid data instead of an empty inventory. The cache only updates with non-empty valid data, and single items are patched in place to avoid reloading everything.

**Cache preload on startup:** inventory, monitors, suppliers, transactions, history and budget are loaded when the server starts, so the first user does not wait.

## Sample (simplified)

```python
import time

LIMIT = {"requests": [], "max_per_minute": 40, "min_interval": 1.5}
CACHE = {}

def under_limit(now): return len([t for t in LIMIT["requests"] if now - t < 60]) < LIMIT["max_per_minute"]

def wait_gap(now): return max(0, LIMIT["min_interval"] - (now - max(LIMIT["requests"], default=0)))

def update_cache(key, data): CACHE[key] = data if data else CACHE.get(key, [])
```

## Notes

Credentials, environment files, supplier data and company-specific details are not part of this repository. Metrics are not claimed here because I have not measured them.
