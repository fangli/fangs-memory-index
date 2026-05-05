# Source Record Schema

Source records are Markdown files under `data/sources/`.

## Frontmatter

```yaml
id: source.<stable-id>
kind: email | chat | file | web | session | manual | calendar | custom
title: string
created_at: ISO-8601
observed_at: ISO-8601
source_spec: source_id from workspace/source-specs/*.md
origin:
  type: message | web | file | session | manual | custom
  id: string
  url: string | null
trust: primary | secondary | assistant-derived | mixed
confidence: 0.0-1.0
entities: []
tags: []
coverage:
  start: date | null
  end: date | null
freshness:
  valid_until: date | null
  refresh_after: date | null
hash: content hash
```

## Body

- `## Summary`
- `## Extracted text or evidence`
- `## Source metadata`
- `## Derived atoms`
