---
name: Ingest purchase orders into CashFlo and reconcile errors
description: >-
  Push a batch of purchase orders into the CashFlo platform via the Data
  Ingestion API, then retrieve and reconcile any per-record errors for the batch.
api: openapi/cashflo-data-ingestion-openapi.json
operations:
- POST /v1/ingest/purchase-orders
- GET /v1/ingest/errors
generated: '2026-07-18'
method: generated
---

# Ingest purchase orders into CashFlo

Use the CashFlo Data Ingestion API to load purchase orders (POs) into the
platform, then check the batch for record-level failures.

## Authentication

All calls require a JWT in the `Authorization` header (security scheme `jwt`,
an apiKey-style header). Obtain the token from CashFlo and send it on every
request. A missing/invalid token returns `401`.

## Step 1 — Ingest the batch

`POST /v1/ingest/purchase-orders` with a JSON body:

- `buyerOrgId` (required) — the buyer organization the POs belong to.
- `skipDuplicates` (optional bool) — set `true` to make re-submission safe;
  records already present are skipped instead of duplicated. This is the API's
  idempotency mechanism (there is no Idempotency-Key header).
- `purchaseOrders` (required array) — each item carries `purchaseOrderNumber`,
  vendor master numbers, facility codes, amounts, and `purchaseOrderItems[]`
  (material, quantity, `gstRate`, `hsnCode`, amounts).

A successful call returns `202 Accepted` with an `IngestionSuccess` body
containing a `batchId`. Retain the `batchId`. A malformed payload returns `400`.

## Step 2 — Retrieve batch errors

`GET /v1/ingest/errors?batchId={batchId}` (batchId required) returns a
`PurchaseOrderErrorsResponse` — a list of `PurchaseOrderError` records
(`status`, `purchaseOrderNumber`, `buyFromVendorMasterNumber`,
`sourceCreationDate`). An unknown batchId returns `404`.

## Step 3 — Reconcile and re-submit

Fix the rejected POs and re-submit them in a new batch with
`skipDuplicates: true` so the accepted records are not duplicated. Repeat until
the error list is empty.

## Conventions and errors

- Versioning: uri-path, `v1`.
- Multi-tenancy: every record is scoped by `buyerOrgId`.
- Errors: `400` bad payload, `401` unauthorized, `404` unknown batch,
  `500` server error. See `errors/cashflo-problem-types.yml`.
