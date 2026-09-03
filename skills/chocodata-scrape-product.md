---
name: chocodata-scrape-product
description: Get structured JSON for a single product, app, video, or post from a supported site via the Chocodata Scraper API.
api: Chocodata Scraper API
operations: [getProduct, getAppStoreProduct, getGooglePlayProduct, getRedditPost, getYouTubeTranscript, getYouTubeComments, getAppStoreReviews, scrape]
generated: '2026-09-03'
method: generated
source: openapi/chocodata-openapi.json + https://chocodata.com/docs
---

# Scrape a single item with Chocodata

Fetch one item as structured JSON from any of the 237 supported sites.

## Steps

1. Authenticate with the `api_key` **query parameter** only — `Authorization: Bearer` and `X-API-Key` both return 401. Production keys start with `asa_live_`.
2. For sites with a dedicated operation, call it directly:
   - `getProduct` — `GET /api/v1/amazon/product?api_key=KEY&query=<ASIN|ISBN>` (or `url=`); add `domain=de` etc. to localise price/currency.
   - `getAppStoreProduct`, `getAppStoreReviews`, `getGooglePlayProduct`, `getYouTubeTranscript`, `getYouTubeComments`, `getRedditPost` follow the same shape with their own identifier params.
3. For every other site/resource, use the generic `scrape` operation — `GET /api/v1/{site}/{resource}` — with that endpoint's **real parameter names**: `walmart/product` takes `url` or `id` (rejects `query`), `bing/search` takes `q`. The full parameter reference per endpoint is at https://chocodata.com/scraper-api.
4. Handle errors from the `{error, message, docs_url, request_id}` envelope. Retry only retryable codes (`408 upstream_timeout` up to 3x with 2s backoff, `429 rate_limited` respecting `Retry-After`, `422`/`502` with 60s+ delay). Never retry `400`, `401`, `404`, `501` — `404 item_not_found` means the item is delisted; drop it.
5. Watch spend: every successful (2xx) response costs 5 credits (`Asa-Cost` header); non-2xx responses are always free. `Asa-Concurrency` and `Asa-Rps` report headroom against your plan's ceilings.
