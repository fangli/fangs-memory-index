# Runbook: Initialize Knowledge Library

Purpose: prepare local generated folders and validate docs. Do not change runtime config or create scheduled jobs in this runbook unless the operator explicitly extends the task.

## Guardrails

- **Role:** setup validation runbook.
- **Mutation:** may create missing generated folders under `data/` only.
- **Allowed writes:** `data/sources/`, `data/atoms/`, `data/views/`, `data/manifests/`, `data/indexes/`, `data/reports/` directory creation.
- **Do not:** edit runtime config, create scheduled jobs, fetch external sources, or modify `system/`.
- **Before acting:** confirm `knowledge_base_dir` resolves to the repository root.
- **After acting:** report which folders were checked or created.
- **Failure mode:** if `knowledge_base_dir` is unclear, stop and ask.

## Steps

1. Read root `AGENTS.md`, `system/AGENTS.md`, `system/GUARDRAILS.md`, `system/SPEC.md`, and `system/RULES.md`.
2. Ensure generated folders exist:

```bash
mkdir -p "$knowledge_base_dir"/data/{sources,atoms,views,manifests,indexes,reports}
```

3. Validate local source specs and examples:

```bash
find "$knowledge_base_dir"/workspace/source-specs -maxdepth 1 -name '*.md' -print | sort
find "$knowledge_base_dir"/system/examples/source-specs -maxdepth 1 -name '*.md' -print | sort
```

4. Validate taxonomy specs:

```bash
find "$knowledge_base_dir"/system/taxonomy/atom-kinds -maxdepth 1 -name '*.md' -print | sort
```

5. Report status to the operator.

## Not included

- No scheduled job creation.
- No runtime config edit.
- No retrieval index rebuild.
- No external source fetch.
