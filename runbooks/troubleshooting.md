# Runbook: Troubleshooting

## Search returns docs/templates instead of user knowledge

Likely cause: project root was indexed.

Fix: remove `knowledge_base_dir` from retrieval paths; index only selected generated output folders under `library/`.

## Agenda misses known items

Check:

1. Was the source collected?
2. Was an `event` or `task` atom extracted?
3. Does the atom have correct time fields?
4. Is the view stale?
5. Was the retrieval index rebuilt after generation?

## Duplicate atoms

Improve dedupe using source id, normalized title, date/time, entities, and text hash. Record skipped duplicate IDs in `library/manifests/`.

## Too many stale answers

Add or tighten `refresh_after`, `valid_until`, and source-specific freshness defaults.

## New source requirement appears repeatedly

Create a new local `sources/<name>.md` spec. Do not add one-off instructions to random docs.
