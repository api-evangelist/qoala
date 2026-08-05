---
name: Create and refresh a Qoala Enterprise session
description: Exchange an email and security code for a JWT access token and refresh token on the Qoala for Enterprise platform, then keep it alive.
api: openapi/qoala-authentication-api-openapi.yml
operations:
  - createSession
  - refreshSession
generated: '2026-08-05'
method: generated
source: openapi/qoala-authentication-api-openapi.yml
---

# Create and refresh a Qoala Enterprise session

This is the **platform** session API, not partner API authentication. Partner integration calls are
authenticated with the `x-api-key` header and do not use these tokens.

## Steps

1. **Create the session.** `createSession` — `POST /v2/sessions` with `email` and `securityCode`.
   Both the `priority` and `x-captcha-token` headers are **required**. The security code is sent to
   the user's email; this is a passwordless code exchange, not a password login.
2. **Read the response.** `SessionResponse.data` returns `token` (JWT access), `refresh` (JWT
   refresh) and the full `user` object, which carries the user's role, permissions and the
   organization profile.
3. **Refresh before expiry.** `refreshSession` — `POST /v2/sessions/refresh` with the `refresh`
   token. The `priority` header is required; `x-captcha-token` is not.

## Rules

- The `user.organization.apiKey` object in the session response contains partner credential material
  including a `plainSecret` field. Never log, echo, cache in plaintext, or surface this to an end
  user or model context.
- These two operations are the only ones published on the Authentication API, and it is the only
  Qoala specification that does **not** declare the `ApiKeyAuth` security scheme.
