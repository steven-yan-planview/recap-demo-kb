---
title: "PDF-1046: Permission Batching in Tenant Resolution"
type: note
permalink: pdf-1046-permission-batching
tags:
  - component/auth
  - type/story
  - area/performance
  - recorded_at: 2026-06-12
---

# Permission Batching in Tenant Resolution

Replaced the N+1 query pattern in tenant permission resolution middleware with request-scoped batching to reduce p99 latency from 800ms+ to under 100ms.

## Problem

The original middleware called `check_permission(tenant_id, resource_id)` once per resource per request. An endpoint returning 50 resources issued 50 separate SQL queries, causing cascading latency spikes.

## Solution: Request-Scoped PermissionService

Implemented `PermissionService` as a FastAPI request-scoped dependency (`Depends(get_permission_service)`) that accumulates all resource IDs in a single request, deduplicates them, and issues one batched SQL query using parameterised `IN` clauses.

### Mechanics

The `check_all()` method:
1. Accepts an iterable of resource IDs
2. Deduplicates against the in-request cache
3. Batches only uncached IDs into a single SQL query using SQLAlchemy `bindparam(..., expanding=True)`
4. Memoises results for the request lifetime

Repeated calls within the same request are free (cache hit). SQL injection is prevented by parameterised queries.

### Error Handling

On database failure, raises `PermissionStoreUnavailable`. This allows endpoints returning sensitive data to fail-closed, while read-only endpoints can opt for fail-open. The caller explicitly chooses behaviour based on endpoint sensitivity; no default hiding of errors.

### Prerequisites

`AuthMiddleware` must have set `request.state.tenant` before the service is invoked. No public API contract change. Integration and regression tests added.

## Observations

- [decision] Request-scoped dependency injection ensures per-request cache never leaks across requests (PDF-1046, 2026-06-12) #isolation
- [decision] Permission store unavailability is raised rather than silenced; callers must explicitly choose fail-open or fail-closed behaviour based on data sensitivity (PDF-1046, 2026-06-12) #error-handling
- [fact] SQL query filters on `expires_at > CURRENT_TIMESTAMP()` to respect permission expiry; this check happens at query time, not cache time (PDF-1046, 2026-06-12) #correctness
- [gotcha] Per-request memoisation is safe only within a single request; expiry within a single HTTP handler is rare but possible in long-running requests (PDF-1046, 2026-06-12) #safety
- [gotcha] Cache key is resource_id only; if the same endpoint is called with different tenant_ids in the same request, separate `PermissionService` instances are created (one per unique tenant context), so there is no cross-tenant cache collision (PDF-1046, 2026-06-12) #multi-tenant
- [pattern] Request-scoped dependency injection in FastAPI to accumulate and batch external lookups; applicable to any per-resource resolution that can be batched, not just permissions (PDF-1046, 2026-06-12) #reusable
- [pattern] Parameterised SQL `IN` clauses with SQLAlchemy's `expanding=True` for safe batch queries (PDF-1046, 2026-06-12) #reusable
- [pattern] Fail-closed vs fail-open error handling strategy: raise a specific exception and let the caller decide the failure mode based on endpoint sensitivity (PDF-1046, 2026-06-12) #reusable
- [pattern] Per-request memoisation as a general pattern for reducing round-trips to slow stores (DB, cache, external API) within a single handler (PDF-1046, 2026-06-12) #reusable

## Relations

- relates_to [[Auth Middleware]]
