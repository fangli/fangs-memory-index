# View Record Schema

Views are generated Markdown files under `data/views/`.

## Frontmatter

```yaml
id: view.<stable-id>
kind: view
title: string
generated_at: ISO-8601
query_shape: agenda | open_loops | recent_changes | expiring_resources | entity_digest | custom
range:
  start: date | null
  end: date | null
atom_refs: []
source_refs: []
```

## Body

- Keep views concise.
- Group by time/status/entity when useful.
- Cite atom/source IDs.
- State if the view is incomplete or stale.
