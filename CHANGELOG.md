## Changelog

## 2026-05-05

- Created initial documentation-first personal memory index project.
- Defined source/atom/view architecture.
- Added pluggable source spec system.
- Added runbooks for installation, daily ingestion, source updates, querying, reindexing, and troubleshooting.
- Generalized project for arbitrary AI-agent runtimes.
- Added SQLite deterministic index schema (`system/schemas/index-schema.md`).
- Added reconciliation rules for correction detection and resolution (`system/rules/reconciliation.md`).
- Added reconciliation examples (`system/examples/reconciliation-examples.md`).
- Updated atom record schema with `retrieval_status`, `observed_at` (required), and correction chain fields.
- Updated daily ingestion runbook with reconciliation pass and SQLite index update step.
- Updated SPEC.md retrieval strategy and correction model.
- Updated GUARDRAILS.md and RULES.md with reconciliation and correction rules.
- Updated SCHEMAS.md index.

## 2026-05-05 (rev 2) — Reconciliation hardening & indexing agent entry point

### Fixes
- Fixed time overlap check: proper range overlap (`time_start <= end_effective AND end_effective >= time_start`) instead of `BETWEEN` on `time_start` alone.
- Fixed duplicate step numbering in answer-from-knowledge runbook.
- Removed non-existent `time.date` field from event kind; clarified mapping to `time.start`.
- Clarified resource coverage uses `valid_from`/`valid_until` (not a separate `coverage` field).

### Additions
- `body_hash` (SHA-256) field added to atom schema and SQLite index for fast deduplication.
- `entity_aliases` table added to SQLite schema for normalized entity matching.
- `resource_type` column added to SQLite schema (used in resource correction matching).
- Entity normalization rules: lowercase, trim, collapse whitespace, alias expansion.
- Explicit per-kind historical transition criteria (event, task, resource, claim, etc.).
- Correction chain depth limit (5 hops retrieval, flatten at depth 4 during indexing).
- Per-kind candidate queries: events/tasks use time overlap, resources use coverage overlap, timeless kinds use entity-only matching.
- Observations explicitly skip correction detection (they coexist).
- Retrieval-time filtering guidance: prefer `retrieval_status = 'active'` filter when available.
- New examples: deduplication (3b), chain flattening (5), historical transition (6), ambiguous match (7).

### Indexing agent entry point
- Daily ingestion runbook now has a **Reading Order** section: 6 phases, 15 numbered steps, covering system → data model → reconciliation → taxonomy → source specs → examples.
- SQLite creation is an explicit precondition (step 0).
- Workflow steps include inline SQL for dedup, candidate queries, view generation.
- Structured output report format with correction details and ambiguity log.
