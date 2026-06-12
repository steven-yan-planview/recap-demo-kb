---
title: Auth Middleware
type: note
permalink: auth-middleware
tags:
  - component/auth
  - area/security
---

# Auth Middleware

## Current Approach

JWT validation middleware sits at the FastAPI middleware layer (`src/auth/middleware.py`). Every request must carry a `Bearer` token in the `Authorization` header.

Tokens are verified against a public key fetched from the configured JWKS endpoint at startup and cached in memory. Key rotation triggers a cache refresh via a background task.

## Token Claims Used

| Claim | Usage |
|-------|-------|
| `sub` | User identity, logged to audit trail |
| `tenant` | Injected as `_tenant_id` on all Snowflake writes |
| `scope` | Checked against route-level permission requirements |
| `exp` | Standard expiry enforcement |

## Prior Auth Approach

Before Sprint 10, auth used a shared API key per tenant passed in the `X-API-Key` header. Removed because key rotation required service restarts and there was no per-request identity for auditing. (PDF-155, 2026-03-18)

## Pending

OAuth 2.0 PKCE flow for interactive clients was planned but not yet implemented as of Sprint 11. Tracked as PDF-203.
