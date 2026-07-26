---
name: Ingest records and track the batch
description: Submit customer-signal records to the Evermuse Ingestion API and poll the batch to completion.
api: openapi/evermuse-ingest-v1-openapi-original.yml
operations: [ingestRecords, getBatchStatus]
---

# Ingest records and track the batch

Use this skill to send customer evidence (emails, calls, meeting notes, conversations,
documents) into the Evermuse data lake and confirm it processed.

## Auth
- Send an `x-api-key: em_sk_...` header on every request. The key needs the `api:write`
  permission or you will get `403`. Never expose the key client-side.

## Steps
1. Build each record against the Integration Envelope: required fields are `_type`,
   `_schema_version` (`"1.0.0"`), `_event_at` (ISO-8601 UTC), `_vendor_ids` (≥1),
   `_product_id`, `_project_ids` (≥1), `_nature` (`evidence`|`guidance`|`context`), and `data`.
   All records in one request must share the same `_type` and `_product_id`; max 1,000 records, max 5 MB.
2. Call `ingestRecords` — `POST /api/v1/ingest`. Body may be a JSON array, a `{ "records": [...] }`
   wrapper, or `application/x-ndjson` for high volume. Set an `Idempotency-Key` header when
   retrying so a duplicate submission returns the original response instead of reprocessing.
3. Read the `202` response: keep `batchId`, and check `acceptedCount`/`rejectedCount`. If
   `rejectedCount > 0`, fix the records named in `errors[]` (each has `index`, `code`, `message`) and resend.
4. Call `getBatchStatus` — `GET /api/v1/ingest/batches/{batchId}` — and poll until `status`
   is `COMPLETE` (or `FAILED`). Lifecycle: `LANDING → LANDED → PROCESSING → COMPLETE`.

## Rules
- Deduplication key is `_type` + `_vendor_ids` + `_event_at`; re-sending overwrites rather than duplicates.
- On `429`, back off for the `retryAfter` seconds returned in the body.
- Reference attachments by `vendor_url`; Evermuse fetches them asynchronously after the batch lands.
