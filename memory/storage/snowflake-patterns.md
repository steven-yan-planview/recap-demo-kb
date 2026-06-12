---
title: Snowflake Patterns
type: note
permalink: snowflake-patterns
tags:
  - component/snowflake
  - area/storage
---

# Snowflake Patterns

## Connection Pool

The service uses `snowflake-connector-python` with a simple in-process connection pool (`src/storage/connection_pool.py`). Pool is initialized at startup with `min_size=2, max_size=10`.

Connections are validated before checkout with a `SELECT 1` probe. Stale connections (idle > 5 minutes) are evicted.

## Write Pattern

All writes use parameterized `executemany` to batch records within a single request. Batch size is capped at 500 rows to stay within Snowflake's recommended per-statement row count.

## RLS Tagging

Every row written includes a `_tenant_id` column set from the authenticated JWT `tenant` claim. Snowflake row-level security policies filter on this column at query time.

## Known Issues

- Pool eviction under high concurrency was unreliable prior to PDF-202 (2026-06-05); connections leaked when acquire timed out mid-eviction cycle.
