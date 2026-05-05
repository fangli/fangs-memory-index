# Changelog

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
