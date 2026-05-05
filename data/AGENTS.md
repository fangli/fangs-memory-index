# data/AGENTS.md — Generated User Knowledge

This folder contains generated user knowledge and indexes.

## Guardrails

- **Role:** generated user knowledge guide.
- **Mutation:** generated-only via approved runbooks.
- **Allowed writes:** `data/sources/`, `data/atoms/`, `data/views/`, `data/manifests/`, `data/indexes/`, `data/reports/`.
- **Do not:** write static instructions, local source specs, credentials, or framework changes here.
- **Before acting:** resolve and verify every write path stays under `data/`.
- **After acting:** report files created/updated, duplicates skipped, and unresolved items.
- **Failure mode:** if provenance is missing, create or reference a source record first; do not create orphan atoms.

Agents may write here when executing approved runbooks such as daily ingestion or manual import.

Do not hand-edit generated files unless a runbook explicitly says to repair or migrate them.

Recommended future retrieval paths are selected subfolders here, not the repository root.

## Subfolders

- `sources/` normalized source records.
- `atoms/` extracted reusable atom records.
- `views/` generated rollups.
- `manifests/` seen IDs, hashes, ingestion state.
- `indexes/` deterministic local indexes such as SQLite.
- `reports/` librarian reports.
