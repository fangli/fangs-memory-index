# Atom Record Schema

Atom records are Markdown files under `data/atoms/`. Multiple atoms may live in one daily batch file.

## Required fields

```yaml
id: atom.<stable-id>
kind: event | task | claim | preference | decision | resource | procedure | observation
created_at: ISO-8601
updated_at: ISO-8601
source_refs: []
confidence: 0.0-1.0
entities: []
tags: []
```

## Common optional fields

```yaml
time:
  start: date-time | date | null
  end: date-time | date | null
  due: date-time | date | null
  expression: string | null
freshness:
  observed_at: ISO-8601 | null
  valid_from: date | null
  valid_until: date | null
  refresh_after: date | null
status: open | done | blocked | maybe | unknown
```

## Body

Use concise prose optimized for memory search. Include enough context for standalone retrieval.
