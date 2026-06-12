---
title: New Engineer Guide
type: note
permalink: new-engineer-guide
tags:
  - area/onboarding
---

# New Engineer Guide

## Accounts to Request

- AWS IAM role: `data-platform-dev` (request via IT portal)
- Snowflake: `DATA_PLATFORM_DEV` database, `INGESTION_ROLE` (request via #data-platform Slack)
- GitHub: `steven-yan-planview` org membership

## Local Setup

1. Clone `recap-demo-app` and `recap-demo-kb`
2. Copy `.env.example` to `.env`, fill in dev credentials
3. `pip install -e ".[dev]"` in `recap-demo-app`
4. `pytest tests/` — all should pass against dev Snowflake

## Key Contacts

- Platform lead: @alex
- Snowflake admin: @priya
- On-call rotation: #data-platform-oncall

## First Week Tips

- Read [[System Overview]] and [[Ingestion Pipeline]] before touching the pipeline code
- All PRs require branch name `PDF-{ticket-id}-{description}` — CI checks this
- State-changing PRs need a Snowflake migration ticket before merge
