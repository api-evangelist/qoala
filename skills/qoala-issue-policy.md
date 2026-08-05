---
name: Issue a Qoala insurance policy (asynchronous)
description: Create a policy from a quotation on any Qoala product line, then confirm issuance via polling or the partner callback webhook.
api: openapi/qoala-logistic-insurance-openapi.yml
operations:
  - createPolicy
  - detailPolicy
generated: '2026-08-05'
method: generated
source: openapi/*.yml + https://docs.qoala.app/reference/api-integration
---

# Issue a Qoala insurance policy

Qoala policy creation is **asynchronous**. The create call only acknowledges receipt; the policy
is confirmed later. Never treat the create response as proof the policy exists.

## Before you start

- Authenticate with the partner API key in the `x-api-key` header. Keys are issued by Qoala
  Engineering during onboarding — there is no self-service key. See `authentication/qoala-authentication.yml`.
- Pick the host for your environment: `https://api-staging.qoala.app` (staging),
  `https://api.uat.qoala.app` (UAT), `https://api.qoala.app` (production).
- Generate a `partner_transaction_number` yourself and keep it unique across your transactions.
  It is your only handle on the request before Qoala returns its own identifiers.

## Steps

1. **Create the policy.** `POST /api/v2/quotation/quotes` with the product payload for your line of
   business. Only the Logistic Insurance specification names this operation (`createPolicy`); on the
   other twelve product specifications the same path and method are published **without an
   operationId**, so address it by method + path.
2. **Read the acknowledgement.** A `200`/`202` returns a `quotation_number` (`QS-` prefixed). This
   acknowledges receipt only — issuance has not completed and no policy number exists yet.
3. **Wait for the result.** Prefer the callback: Qoala POSTs the full policy detail to your
   configured callback URL when issuance completes. See `asyncapi/qoala-webhooks.yml`.
   If you have no callback endpoint, poll `detailPolicy`
   (`GET /api/v2/quotation/quotes/partner/status`) using the quotation number.
4. **Branch on status.** `POLICY_ACTIVE` means cover is in force. `ISSUING_POLICY` means keep
   waiting. `DATA_VERIFICATION_NEEDED` means the object must be verified before cover starts — for
   gadget products continue with `activationPolicy`. `POLICY_REJECTED_INSURANCE` is terminal.
5. **Store the identifiers.** Persist `quotation_number`, `policy_number` and `transaction_number`
   against your `partner_transaction_number`.

## Rules

- Do not retry a create call just because it was slow — you have no idempotency key on this path,
  only your own `partner_transaction_number`. Reconcile with `detailPolicy` before resubmitting.
- Insurer communication is internal to Qoala and never blocks the partner exchange.
- Errors use Qoala's own envelope (`status`, `data`, `message`, `code`, `error_code`), not
  RFC 9457. See `errors/qoala-problem-types.yml`.
