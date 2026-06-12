---
title: Rate Limiting
type: note
permalink: rate-limiting
tags:
  - component/api
  - area/api
---

# Rate Limiting

## Status

Rate limiting on the ingestion API was identified as a requirement in Sprint 11 planning but not yet implemented. Tracked as PDF-201.

## Planned Approach

- Per-tenant rate limiting keyed on JWT `tenant` claim
- Sliding window, 1000 req/min default
- 429 response with `Retry-After` header
- Limits configurable per tenant via SSM

## Why Needed

Without per-tenant limits a single misbehaving producer can saturate the Snowflake connection pool, causing 503s for all other tenants.
