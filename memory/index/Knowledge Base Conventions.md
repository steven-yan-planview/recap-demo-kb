---
title: Knowledge Base Conventions
type: guide
permalink: kb-conventions
tags:
  - meta/conventions
---

# Knowledge Base Conventions

This document defines the house style for all notes in this KB. RECAP's WriterAgent reads this file to format generated notes consistently.

## Frontmatter

Every note must have YAML frontmatter:

```yaml
---
title: "PDF-123: Short Descriptive Title"
type: note
permalink: pdf-123-short-descriptive-title
tags:
  - component/access-proxy
  - type/story
  - sprint/sprint-12
  - recorded_at: 2026-06-07
---
```

**Rules:**
- `title` includes ticket key prefix for ticket-derived notes
- `permalink` is kebab-case, lowercase
- `tags` use namespaced format: `component/`, `type/`, `sprint/`, `area/`
- Do not add `recorded_at` manually — RECAP sets it

## Headings

Use **semantic headings** that describe the content, not generic templates.

Good: `## Why We Moved Off SQS`, `## Connection Pool Sizing Decision`
Bad: `## What Was Done`, `## Key Learnings`, `## Summary`

## Body Style

- Minimum words — no filler phrases ("It is worth noting that...")
- Past tense for decisions, present tense for ongoing facts
- Cite evidence: `(PDF-123, 2026-06-07)` when referencing a prior decision
- No emoji

## Wikilinks

Use `[[Note Title]]` syntax to cross-reference other notes. **Only link to notes that exist in this KB.** Do not invent links.

Existing notes you can link to:
- [[Ingestion Pipeline]]
- [[Snowflake Patterns]]
- [[Auth Middleware]]
- [[System Overview]]
- [[Rate Limiting]]

## File Naming

- Story / Epic / Feature: `{ticket-key-lowercase}-{slugified-title}.md`
  - Example: `pdf-203-oauth2-pkce-migration.md`
- Bug / Task: append findings to the relevant component note — do not create a new file

## Write Modes

| Mode | When | Effect |
|------|------|--------|
| FULL | New topic, no prior entry | Create new file |
| DELTA | Prior entry exists, new evidence | Append new sections or observations |
| REPLACE | Prior entry is contradicted or approach changed | Rewrite the affected sections |
