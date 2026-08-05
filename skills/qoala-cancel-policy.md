---
name: Cancel a Qoala policy
description: Look up an issued policy and cancel it, handling the asynchronous status transition.
api: openapi/qoala-gadget-insurance-openapi.yml
operations:
  - detailPolicy
  - cancelPolicy
generated: '2026-08-05'
method: generated
source: openapi/*.yml + https://docs.qoala.app/reference/policy-status
---

# Cancel a Qoala policy

## Steps

1. **Confirm the current state.** `detailPolicy` — `GET /api/v2/quotation/quotes/partner/status`.
   Only a policy that is actually issued can be cancelled; do not attempt cancellation while the
   status is still `ISSUING_POLICY` or `POLICY_WAITING_PAYMENT`.
2. **Cancel.** `cancelPolicy` — `PATCH /api/v2/policies/partner/cancel`. This operation exists with
   the same id on every product specification (bus, credit, credit-life, experience, flight, gadget,
   goods, hotel, logistic, micro-health, train, vehicle).
3. **Confirm.** Re-read with `detailPolicy`, or wait for the policy status callback, until the
   status is `POLICY_CANCELLED`.

## Rules

- `POLICY_EXPIRED` is terminal and is not the same as cancellation — do not cancel an expired policy.
- A `404` means the identifier is unknown to Qoala; re-check which identifier you sent
  (`quotation_number` vs `policy_number`).
- Several product specifications publish a `400` response whose description reads "Unauthorized".
  Treat the numeric status as authoritative, not the description.
