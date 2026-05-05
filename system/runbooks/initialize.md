# Runbook: Initialize Knowledge Library

Purpose: prepare local generated folders and validate docs. Do not change runtime config or create scheduled jobs in this runbook unless the operator explicitly extends the task.

## Steps

1. Read root `AGENTS.md`, `system/AGENTS.md`, `system/SPEC.md`, and `system/RULES.md`.
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
