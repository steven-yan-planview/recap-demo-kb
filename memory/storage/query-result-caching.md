---
title: "PDF-1045: Snowflake Query Result Caching Layer"
type: note
permalink: pdf-1045-snowflake-query-result-caching
tags:
  - component/storage
  - type/story
  - area/performance
  - recorded_at: 2026-06-12
---

# Snowflake Query Result Caching Layer

In-process LRU cache for Snowflake query results that reduces latency for repeated identical queries from ~2s to <1ms on warm hits.

## Cache Architecture

QueryResultCache is an in-process LRU implementation in src/storage/query_cache.py. Cache keys are (sha256(query)[:16], tenant_id) with a configurable maxsize (default 2048 entries) and 5-minute TTL. Query results are immutable within the TTL window.

In-process LRU was chosen over Redis because Redis would add ~1ms network overhead per hit, negating the benefit for sub-millisecond result sets. Per-pod storage is the correct trade-off at current traffic volumes.

## Cache Invalidation

Invalidation is tenant-scoped via invalidate_tenant(tenant_id), called by schema-change event handlers. This prevents serving stale results when the underlying schema changes. The key format is {tenant_id}:{query_hash}, enabling efficient prefix-based bulk eviction.

## TTL and Memory Management

TTL eviction is lazy: expired entries are only removed when accessed, not by a background task. Memory is bounded by maxsize only; expired entries may remain in memory until accessed or evicted due to capacity pressure. An optional background sweep task can run periodic cleanup if memory pressure becomes a concern.

## Observability

Cache performance is exposed via Prometheus metrics: hit and miss counters are stored as object properties and scraped directly by Prometheus.

## Observations

- [decision] In-process LRU cache instead of Redis to avoid ~1ms network overhead per hit. Per-pod storage is intentional for latency reasons. (PDF-1045, 2026-06-12) #performance
- [decision] Cache keys are (sha256(query)[:16], tenant_id) with 5-minute TTL and configurable maxsize (default 2048 entries). (PDF-1045, 2026-06-12) #design
- [decision] TTL eviction is lazy (only on read), not background-driven by default. Memory bounded by maxsize; expired entries remain until accessed. (PDF-1045, 2026-06-12) #memory
- [decision] Invalidation is tenant-scoped via invalidate_tenant(tenant_id), called by schema-change event handlers. (PDF-1045, 2026-06-12) #correctness
- [gotcha] Cache is per-pod, so query results are not shared across instances. Cache warm-up happens independently per pod. (PDF-1045, 2026-06-12) #distributed-systems
- [gotcha] TTL eviction is lazy, not background-driven by default. Expired entries may consume memory until accessed. (PDF-1045, 2026-06-12) #memory
- [pattern] Thread-safe LRU using OrderedDict with explicit lock management. (PDF-1045, 2026-06-12) #reusable
- [pattern] Dual-counter metrics (hits/misses) exposed as object properties for Prometheus scraping. (PDF-1045, 2026-06-12) #reusable
- [pattern] Prefix-based tenant invalidation with key format {tenant_id}:{query_hash} for efficient bulk eviction. (PDF-1045, 2026-06-12) #reusable
- [fact] Reduces latency for repeated identical queries from ~2s to <1ms on warm hits. (PDF-1045, 2026-06-12) #performance

## Relations

- relates_to [[Snowflake Patterns]]
- relates_to [[System Overview]]
