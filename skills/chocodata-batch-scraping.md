---
name: chocodata-batch-scraping
description: Submit up to 1,000 scrape jobs asynchronously with webhook or polling delivery via the Chocodata Batch endpoint.
api: Chocodata Scraper API
operations: [createBatch]
generated: '2026-09-03'
method: generated
source: openapi/chocodata-openapi.json + https://chocodata.com/docs/endpoints/batch
---

# Batch scraping with Chocodata

Fire-and-forget bulk scraping for 100+ items; for a handful of items use the sync endpoints instead (batch adds ~60s of overhead).

## Steps

1. Submit with `createBatch` — `POST /api/v1/{site}/batch?api_key=KEY` with JSON body: `endpoint` in `site.resource` form (e.g. `walmart.product`, `universal.get`), `items` (1-1,000 objects, each shaped like that endpoint's query params), and optionally a public https `webhook_url`.
2. **Save `webhook_signature_secret` from the 201 response immediately** — it is returned exactly once and is needed to verify webhook authenticity.
3. **Confirm before submitting large batches**: each successful item bills 5 credits (a full 1,000-item batch can cost 5,000 credits) and there is no documented cancel operation once submitted. Failed items are free.
4. Receive results either way:
   - Webhook: on completion Chocodata POSTs the full results to `webhook_url` with headers `X-ASA-Batch-Id`, `X-ASA-Event: batch.completed`, and `X-ASA-Signature: sha256=<hmac-hex>`. Verify with a timing-safe HMAC SHA-256 comparison of the raw body against the saved secret.
   - Polling: `GET /api/v1/{site}/batch/{id}?api_key=KEY` returns `status` (`pending | running | complete | failed`), progress counts, `credits_charged`, and per-item `results`.
5. Expect a 1,000-item batch to complete in ~10-12 minutes (the worker runs every 60s, up to 100 items per run). A long poll holds one concurrency slot; batch items themselves do not count against your concurrency ceiling.
