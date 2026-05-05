# Runbook: Daily Knowledge Ingestion

Purpose: process the previous local day into source records, atoms, and views.

This is the runbook a future scheduled job should execute. It is not a scheduled job by itself.

## Inputs

- Local timezone: operator-configured timezone.
- Date range: previous local calendar day unless specified otherwise.
- Source specs: all enabled local `../sources/*.md` specs.
- Category specs: all `../entities/*.md` specs.

## Workflow

1. Read `../AGENTS.md`, `../RULES.md`, `../SPEC.md`.
2. Read `../sources/AGENTS.md` and all enabled local source specs.
3. For each enabled source spec:
   - collect candidate records for the date range;
   - skip duplicates using manifest/hash;
   - write normalized source records under `../library/sources/`;
   - extract atoms under `../library/atoms/`.
4. Dedupe atoms by normalized text, time, entity, and source refs.
5. Generate/update views:
   - `../library/views/agenda-next-14-days.md`
   - `../library/views/open-loops.md`
   - `../library/views/recent-changes.md`
   - `../library/views/refresh-needed.md`
6. Write report under `../library/reports/YYYY-MM-DD.md`.
7. If installation has enabled retrieval indexing, follow `reindex.md`; otherwise only report that reindex is pending.

## Extraction policy

Extract atom kinds based on `entities/`, not topics.

## Output report format

```text
# Daily Knowledge Report — YYYY-MM-DD

- Sources scanned:
- Source records written:
- Atoms written:
- Views updated:
- Duplicates skipped:
- Needs follow-up:
```
