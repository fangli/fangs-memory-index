# Runbooks Index

Read root `AGENTS.md`, `system/AGENTS.md`, and `system/RULES.md` first.

## Core runbooks

- `system/runbooks/initialize.md` — create/check local generated folders and dry-run setup.
- `system/runbooks/daily-ingestion.md` — daily librarian workflow.
- `system/runbooks/add-source.md` — add/update/delete pluggable local source specs.
- `system/runbooks/answer-from-knowledge.md` — retrieval strategy for user questions.
- `system/runbooks/reindex.md` — retrieval indexing procedure after explicit approval.
- `system/runbooks/troubleshooting.md` — common failures and fixes.

## Source specs

Local source ingestion behavior is defined in `workspace/source-specs/*.md`. Generic examples live under `system/examples/source-specs/`.

## Taxonomy

Atom kinds and category semantics are defined in `system/taxonomy/atom-kinds/*.md`. See `system/taxonomy/AGENTS.md`.
