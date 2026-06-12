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

Rate limiting on the ingestion API was identified as a requirement in Sprint 11 planning. Initial sliding window implementation has been replaced with a Redis-backed token bucket deployed in production.

## Planned Approach

- Per-tenant rate limiting keyed on JWT `tenant` claim
- Sliding window, 1000 req/min default
- 429 response with `Retry-After` header
- Limits configurable per tenant via SSM

## Why Needed

Without per-tenant limits a single misbehaving producer can saturate the Snowflake connection pool, causing 503s for all other tenants.

## Redis Token Bucket Implementation

### Algorithm

Token bucket algorithm refills at a configured rate and allows bursts up to a capacity limit. On each request, the implementation attempts to consume one token. If tokens are available, the request proceeds; otherwise a 429 is returned. The refill rate (tokens per minute) and capacity are configurable per tenant.

### TOCTOU Prevention

Multi-replica deployments require atomic refill and consume operations to prevent a single tenant from bypassing limits by hitting multiple pods simultaneously. The implementation uses a Lua script executed on Redis that reads the current token count, computes refill based on elapsed time, consumes one token, and writes the new count back—all in a single round-trip. This eliminates time-of-check-time-of-use races across replicas.

### Failure Handling

Rate limiting is quality-of-service, not a security gate. Redis unavailability does not block ingestion. On Redis error (connection failure, timeout), the limiter fails open: the request is allowed to proceed without rate limit enforcement. This preserves ingestion availability during Redis incidents at the cost of temporarily elevated traffic.

### Configuration

Capacity (max tokens), rate_per_minute (refill rate), and TTL are configurable. TTL is set to 2 times the refill window (120 seconds) so that per-tenant token buckets expire naturally when a tenant is inactive, freeing memory.

## Relations

- [[Ingestion Pipeline]]
- [[System Overview]]

## Observations

- Prior in-process SlidingWindowRateLimiter allowed N times the configured limit in N-pod deployments because each pod maintained independent counters. Distributed token bucket eliminates this vulnerability. (PDF-1044, 2026-06-12)
- Token bucket chosen over strict sliding window to tolerate brief legitimate ingestion spikes common at the top of the hour. (PDF-1044, 2026-06-12)
- Lua script refill logic uses server-side Redis time (tonumber(ARGV[3])). Clock skew between client and Redis server could cause unexpected refill timing or token budget drift. (PDF-1044, 2026-06-12)
- Fail-open policy on Redis errors preserves availability for a non-critical QoS feature over strict rate limit enforcement. (PDF-1044, 2026-06-12)
