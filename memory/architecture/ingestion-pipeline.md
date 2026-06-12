---
title: Ingestion Pipeline
type: note
permalink: ingestion-pipeline
tags:
  - component/ingestion
  - area/architecture
---

# Ingestion Pipeline

## Schema Validation

Schemas are defined in `src/ingestion/validators.py` as Pydantic models. Each inbound `POST /ingest/{schema}` is matched against the registered schema. Unknown fields are stripped; missing required fields return 422.

## Transformation

Field normalization runs after validation:
- Timestamps coerced to UTC ISO-8601
- Null string `"null"` coerced to Python `None`
- Numeric strings cast to float if schema type is `number`

## Dead-Letter Queue

Records that fail Snowflake writes are published to an SQS DLQ with a 15-minute visibility timeout. A Lambda consumer retries up to 3 times with exponential backoff before routing to an S3 error bucket.

## Known Constraints

- Max payload size: 1 MB per request
- No batch endpoint yet — each record is a separate POST (PDF-180, 2026-04-12)
- Schema registry is in-process; no remote schema store (tracked for Q3)
