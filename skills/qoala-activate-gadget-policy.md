---
name: Activate a Qoala gadget policy after verification
description: Complete activation for gadget policies whose insured object must be verified before cover starts.
api: openapi/qoala-gadget-insurance-openapi.yml
operations:
  - detailPolicy
  - activationPolicy
generated: '2026-08-05'
method: generated
source: openapi/qoala-gadget-insurance-openapi.yml + https://docs.qoala.app/reference/activationpolicy-1
---

# Activate a Qoala gadget policy

Some insured objects must be verified before cover begins — typically used or second-hand gadgets.
New devices generally do not need this step. Activation is published only on the **Gadget Insurance**
specification.

## Steps

1. **Check whether activation is required.** `detailPolicy` —
   `GET /api/v2/quotation/quotes/partner/status`. A status of `DATA_VERIFICATION_NEEDED` means
   documents or information are outstanding.
2. **Activate.** `activationPolicy` — `POST /api/v2/policies/documents/activation`, supplying the
   verification documents for the insured device.
3. **Confirm.** Re-read with `detailPolicy` until the status is `POLICY_ACTIVE`.

## Rules

- Only policies **not** already in `POLICY_ACTIVE` can be activated.
- Do not tell a customer they are covered until the status reads `POLICY_ACTIVE`.
