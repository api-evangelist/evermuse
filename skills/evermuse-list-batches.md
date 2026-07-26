---
name: Audit recent ingestion batches
description: Page through recently submitted Evermuse batches and inspect their processing status.
api: openapi/evermuse-ingest-v1-openapi-original.yml
operations: [listBatches, getBatchStatus]
---

# Audit recent ingestion batches

Use this skill to review what has been sent into the Evermuse data lake and whether it processed cleanly.

## Auth
- Send an `x-api-key: em_sk_...` header. Reading batch status works with an `api:read` key.

## Steps
1. Call `listBatches` — `GET /api/v1/ingest/batches`. Optional `limit` (default 20, max 100).
   Results come newest-first.
2. Page with the cursor: pass the response's `nextCursor` as the `cursor` query param on the
   next call. Stop when `nextCursor` is `null`.
3. For any batch of interest, call `getBatchStatus` — `GET /api/v1/ingest/batches/{batchId}` —
   to read `status`, `acceptedCount`, `rejectedCount`, `errors[]`, and, when attachments were
   included, the `attachmentDownload*` progress fields.

## Rules
- A `404` from `getBatchStatus` means the batch ID does not exist or belongs to another workspace.
- Treat this surface as read-only; it does not mutate the data lake.
