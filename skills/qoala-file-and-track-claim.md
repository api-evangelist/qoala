---
name: File and track a Qoala claim
description: Create a claim against an existing policy, attach supporting documents via presigned URLs, track status, and cancel if needed.
api: openapi/qoala-claim-api-openapi.yml
operations:
  - createClaim
  - presignCreateClaim
  - presignUploadClaim
  - statusClaim
  - cancelClaim
generated: '2026-08-05'
method: generated
source: openapi/qoala-claim-api-openapi.yml + https://docs.qoala.app/reference/createclaim-1
---

# File and track a Qoala claim

## Before you start

- Send the partner API key in the `x-api-key` header.
- A claim is always made against an existing policy. Qoala validates that the policy is claimable
  and rejects the claim if it is not.

## Steps

1. **Create the claim.** `createClaim` — `POST /api/v2/claims/partner/create`. The response carries
   the `claim_number`. Qoala can also notify you of the outcome via the claim callback.
2. **Get an upload URL for documents.** `presignCreateClaim` — `POST /api/v2/claims/partner/update/presign`
   returns a presigned URL. Upload the document to that URL directly; do not POST file bytes to the
   Qoala API.
3. **Replace a document.** `presignUploadClaim` — `PUT /api/v2/claims/partner/update/presign` for an
   updated version of an already-submitted document.
4. **Track status.** `statusClaim` — `GET /api/v2/claims/partner/status`. Or subscribe to the claim
   status callback, which fires on every change (`asyncapi/qoala-webhooks.yml`). The callback is
   optional but is the only way to learn of an approval or rejection without polling.
5. **Cancel if required.** `cancelClaim` — `PATCH /api/v2/claims/partner/cancel`.

## Claim status vocabulary

`CLAIM_INITIATE` → `QOALA_CLAIM_APPROVE` | `QOALA_CLAIM_REJECT` → `INSURANCE_CLAIM_APPROVE` |
`INSURANCE_CLAIM_REJECT` → `INSURANCE_CLAIM_WAITING_PAID` → `INSURANCE_CLAIM_PAID`.

Qoala approval and insurer approval are **two separate gates**. `QOALA_CLAIM_APPROVE` does not mean
the claim will be paid.

## Stop-loss pools

For finance products backed by a stop-loss pool, read the remaining balance with `getPool` —
`GET /api/v2/products/finance/partner/products/stoploss/{claim_pool_code}`. Check the pool before
submitting a high-value claim.

## Rules

- Retry callbacks Qoala sends you by returning a non-200; Qoala retries up to 8 times with
  exponential backoff. Return 200 only once you have durably recorded the notification.
- Qoala publishes a second, near-identical claim specification (`openapi/qoala-claim-api-alt-openapi.yml`)
  with the same paths and operationIds. Use either; they describe the same surface.
