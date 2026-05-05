# AGENTS.md — Repository Boundary Guide

This repository is split into immutable framework files and mutable local/generated state.

## Guardrails

- **Role:** repository boundary guide.
- **Mutation:** edit only when explicitly changing repository-level behavior.
- **Allowed writes:** none during normal ingestion or indexing.
- **Do not:** use this file as permission to create schedules, edit runtime config, or index the repository root.
- **Before acting:** read `system/GUARDRAILS.md` and the relevant runbook.
- **Failure mode:** if folder permissions or runbook scope are unclear, stop and report.

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
2. `system/GUARDRAILS.md`
3. `system/RULES.md`
4. the relevant runbook under `system/runbooks/`
5. local specs under `workspace/source-specs/` if source ingestion is involved

Never configure the repository root or `system/` as retrieval/indexing paths. Index generated data only, typically selected subfolders under `data/`.
