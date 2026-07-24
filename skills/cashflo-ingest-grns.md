---
name: Ingest goods-receipt notes (GRNs) into CashFlo
description: >-
  Push a batch of goods-receipt notes into the CashFlo platform via the Data
  Ingestion API and track the batch for reconciliation.
api: openapi/cashflo-data-ingestion-openapi.json
operations:
- POST /v1/ingest/grns
- GET /v1/ingest/errors
generated: '2026-07-18'
method: generated
---

# Ingest goods-receipt notes into CashFlo

Load goods-receipt notes (GRNs) into the CashFlo platform so they can be matched
against purchase orders and invoices.

## Authentication

Send a JWT in the `Authorization` header on every request (security scheme
`jwt`). A missing/invalid token returns `401`.

## Step 1 — Ingest the GRN batch

`POST /v1/ingest/grns` with a `GRNRequest` body:

- `buyerOrgId` (required) — the buyer organization.
- `skipDuplicates` (optional bool) — skip entries already present, making
  re-submission safe (payload-level idempotency).
- `grns` (required array) — the goods-receipt note entries, each with its line
  items.

A successful call returns `202 Accepted` with a tracking `batchId`. A malformed
payload returns `400`.

## Step 2 — Check for errors

`GET /v1/ingest/errors?batchId={batchId}` returns any per-record failures for
the batch. Fix and re-submit rejected GRNs with `skipDuplicates: true`.

## Conventions and errors

- Versioning: uri-path, `v1`; tenancy via `buyerOrgId`.
- See `conventions/cashflo-conventions.yml` and
  `errors/cashflo-problem-types.yml`.
