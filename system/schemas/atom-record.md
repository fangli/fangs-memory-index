# Atom Record Schema

Atom records are Markdown files under `data/atoms/`. Multiple atoms may live in one daily batch file.

## Required fields

```yaml
id: atom.<stable-id>
kind: event | task | claim | preference | decision | resource | procedure | observation
retrieval_status: active | corrected | historical
created_at: ISO-8601
updated_at: ISO-8601
observed_at: ISO-8601
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
  valid_from: date | null
  valid_until: date | null
  refresh_after: date | null
status: open | done | blocked | maybe | unknown
```

## Correction fields (set by reconciliation)

```yaml
corrects: atom-id | null        # atom this one replaces
corrected_by: atom-id | null    # atom that replaced this one
corrected_at: ISO-8601 | null   # when correction happened
```

## Body

Use concise prose optimized for memory search. Include enough context for standalone retrieval.

For corrected atoms, the body must begin with a correction notice:

```markdown
> ⚠️ CORRECTED — This atom has been superseded.
> Current version: <new-atom-id> (<brief title>, in <file>)
> Reason: <why>

~~Original content here~~
```

The retrieval agent reads this notice and follows the pointer to the current version.

## retrieval_status semantics

| Value | Meaning | When set |
|-------|---------|----------|
| `active` | Current truth or forward-looking mutable fact | Default for new atoms |
| `corrected` | Superseded by newer version; inline notice points to replacement | Set by reconciliation |
| `historical` | Past fact, true for its time period | Set when time range passes into the past without contradiction |
