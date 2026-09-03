---
name: chocodata-universal-scrape
description: Scrape any URL on the web to structured JSON, clean HTML, or plain text via the Chocodata Universal Web Scraper.
api: Chocodata Scraper API
operations: [universalGet]
generated: '2026-09-03'
method: generated
source: openapi/chocodata-openapi.json + https://chocodata.com/docs/endpoints/universal
---

# Universal web scraping with Chocodata

For any page without a dedicated endpoint — Chocodata handles residential proxies, CAPTCHAs, and anti-bot measures server-side.

## Steps

1. Prefer a dedicated endpoint when one exists (check https://chocodata.com/scraper-api) — dedicated endpoints return parsed, parity-checked structured fields; Universal returns the page generically.
2. Call `universalGet` — `GET /api/v1/universal/get?api_key=KEY&url=<encoded target URL>`.
3. Do not pass `render_js` or `screenshot` — they are reserved roadmap parameters and return `501 not_implemented` (free, but the call fails).
4. Retry semantics are the standard Chocodata rules: back off on `429` per `Retry-After`; `422 blocked_by_target` and `502 target_unreachable` merit a 60s+ delayed retry; only 2xx responses cost credits (5 each).
5. For bulk URL sets, submit through the Batch endpoint with `endpoint: "universal.get"` instead of looping sync calls.
