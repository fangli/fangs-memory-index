# AGENTS.md — Repository Boundary Guide

This repository is split into immutable framework files and mutable local/generated state.

## Folder roles

- `system/` — static framework instructions, schemas, taxonomy, templates, examples, and runbooks. Treat as read-only unless the operator explicitly asks to change the framework.
- `workspace/` — mutable local configuration. Agents may update it when following system runbooks.
- `data/` — generated user knowledge and indexes. Daily ingestion may update this folder.

## Mutation policy

Default permissions for agents:

```text
system/     read-only
workspace/  editable via runbook
data/       editable/generated via runbook
```

Do not create scheduled jobs, change runtime config, rebuild retrieval indexes, or fetch external data unless the operator explicitly asks.

## Required reading

For normal operation, read:

1. `system/AGENTS.md`
2. `system/RULES.md`
3. the relevant runbook under `system/system/runbooks/`
4. local specs under `workspace/source-specs/` if source ingestion is involved

Never configure the repository root or `system/` as retrieval/indexing paths. Index generated data only, typically selected subfolders under `data/`.
