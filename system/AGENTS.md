# system/AGENTS.md — Static Framework Operator Guide

This folder contains static framework instructions for a generic personal memory index.

## Immutability rule

Agents should not modify files under `system/` unless the operator explicitly asks to change the framework itself.

## Before working

Read in this order:

1. root `AGENTS.md`
2. `system/SPEC.md`
3. `system/RULES.md`
4. relevant `system/runbooks/*.md`
5. for local source changes: `workspace/AGENTS.md` and relevant files under `workspace/source-specs/`
6. for atom/category behavior: `system/taxonomy/AGENTS.md` and relevant `system/taxonomy/atom-kinds/*.md`

## Mental model

Use this pipeline:

```text
workspace source specs -> collectors -> data source records -> atom extraction -> deterministic metadata/indexes -> generated views -> retrieval index
```

The subject is arbitrary. A flyer, work message, web page, or chat message may all generate the same atom kinds: `event`, `task`, `claim`, `resource`, etc.

## Output locations

Generated content belongs under `data/`:

- `data/sources/` — normalized source records.
- `data/atoms/` — extracted atom records.
- `data/views/` — generated rollups.
- `data/manifests/` — hashes, seen IDs, ingestion state.
- `data/indexes/` — deterministic local indexes, e.g. SQLite.
- `data/reports/` — ingestion/librarian reports.

## Indexed paths after installation

Future installation should index selected generated output folders, for example:

```text
knowledge_base_dir/data/sources
knowledge_base_dir/data/atoms
knowledge_base_dir/data/views
knowledge_base_dir/data/reports
```

Never index `knowledge_base_dir`, `system/`, or `workspace/` as retrieval paths because they contain instructions, templates, specs, and local configuration rather than generated user knowledge.
