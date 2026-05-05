# Runbook: Troubleshooting

## Guardrails

- **Role:** diagnostic runbook.
- **Mutation:** read-only by default.
- **Allowed writes:** none unless a referenced repair runbook explicitly allows it.
- **Do not:** delete data, rebuild indexes, or edit config as a first response.
- **Before acting:** classify the symptom and inspect relevant manifests/config/docs.
- **After acting:** report diagnosis, evidence, and the safest next repair step.
- **Failure mode:** if diagnosis is uncertain, propose a dry-run check instead of changing state.

## Search returns docs/templates/config instead of user knowledge

Likely cause: project root, `system/`, or `workspace/` was indexed.

Fix: remove those paths from retrieval config; index only selected generated output folders under `data/`.

## Agenda misses known items

Check:

1. Was the source collected?
2. Was an `event` or `task` atom extracted?
3. Does the atom have correct time fields?
4. Is the view stale?
5. Was the retrieval index rebuilt after generation?

## Duplicate atoms

Improve dedupe using source id, normalized title, date/time, entities, and text hash. Record skipped duplicate IDs in `data/manifests/`.

## Too many stale answers

Add or tighten `refresh_after`, `valid_until`, and source-specific freshness defaults.

## New source requirement appears repeatedly

Create a new local `workspace/source-specs/<name>.md` spec. Do not add one-off instructions to random docs.
