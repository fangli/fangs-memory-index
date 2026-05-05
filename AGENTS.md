# AGENTS.md — Knowledge Library Operator Guide

You are operating a local personal memory index for an AI-agent runtime.

## Non-negotiables

1. Treat the project root as `knowledge_base_dir`.
2. Keep project files under `knowledge_base_dir`.
3. Do **not** configure `knowledge_base_dir` itself as a retrieval/search path.
4. Do **not** create scheduled jobs, edit runtime config, rebuild indexes, or fetch external data unless the operator explicitly asks.
5. Prefer runbooks and specs over ad-hoc one-off commands.
6. Add new ingestion requirements by creating/updating/deleting Markdown specs in `sources/`, not by scattering special cases across random files.
7. Organize by information behavior, not by topic folders.

## Before working

Read in this order:

1. `README.md`
2. `SPEC.md`
3. `RULES.md`
4. Relevant `runbooks/*.md`
5. For source changes: `sources/AGENTS.md` and relevant source spec files.
6. For atom/category changes: `entities/AGENTS.md` and relevant category files.

## Mental model

Use this pipeline:

```text
source specs -> collectors -> source records -> atom extraction -> deterministic metadata/indexes -> generated views -> retrieval index
```

The subject is arbitrary. A flyer, work message, web page, or chat message may all generate the same atom kinds: `event`, `task`, `claim`, `resource`, etc.

## Output locations

Generated content belongs under `library/`:

- `library/sources/` — normalized source records.
- `library/atoms/` — extracted atom records.
- `library/views/` — generated rollups.
- `library/manifests/` — hashes, seen IDs, ingestion state.
- `library/indexes/` — deterministic local indexes, e.g. SQLite.
- `library/reports/` — ingestion/librarian reports.

## Indexed paths after installation

Future installation should index selected generated output folders, for example:

```text
knowledge_base_dir/library/sources
knowledge_base_dir/library/atoms
knowledge_base_dir/library/views
knowledge_base_dir/library/reports
```

Never index the project root because it contains instructions, templates, and specs that are not user knowledge.
