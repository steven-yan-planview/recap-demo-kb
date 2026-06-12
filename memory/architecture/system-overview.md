---
title: System Overview
type: note
permalink: system-overview
tags:
  - area/architecture
  - component/platform
---

# System Overview

The Data Platform ingestion service sits between producer systems and Snowflake. Producers POST records to the REST API; the ingestion pipeline validates, transforms, and writes to Snowflake via a managed connection pool.

## Components

- **API layer** — FastAPI, handles auth, rate limiting, request routing
- **Ingestion pipeline** — Schema validation, field normalization, DLQ for failures
- **Snowflake client** — Connection pool, retry with exponential backoff, RLS tagging

## Data Flow

```
Producer → POST /ingest/{schema}
         → Auth middleware (JWT validation)
         → Rate limiter
         → Schema validator
         → Transformer
         → Snowflake writer
         → 202 Accepted
```

Failed records go to the dead-letter queue (DLQ); a separate consumer retries with backoff.

## Deployment

Runs as a containerized service on ECS Fargate. One task definition per environment (dev, staging, prod).
Config injected via SSM Parameter Store at runtime.
