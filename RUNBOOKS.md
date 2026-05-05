# Runbooks Index

Read `AGENTS.md` and `RULES.md` first.

## Core runbooks

- `runbooks/initialize.md` — create/check local generated folders and dry-run setup.
- `runbooks/daily-ingestion.md` — daily librarian workflow.
- `runbooks/add-source.md` — add/update/delete pluggable source specs.
- `runbooks/answer-from-knowledge.md` — retrieval strategy for user questions.
- `runbooks/reindex.md` — memory indexing procedure after explicit approval.
- `runbooks/troubleshooting.md` — common failures and fixes.

## Source specs

Source ingestion behavior is defined in `sources/*.md`. See `sources/AGENTS.md`.

## Entity/category specs

Atom kinds and category semantics are defined in `entities/*.md`. See `entities/AGENTS.md`.
