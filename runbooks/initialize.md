# Runbook: Initialize Knowledge Library

Purpose: prepare local generated folders and validate docs. Do not change runtime config or create scheduled jobs in this runbook unless the operator explicitly extends the task.

## Steps

1. Read `../AGENTS.md`, `../SPEC.md`, and `../RULES.md`.
2. Ensure generated folders exist:

```bash
mkdir -p "$knowledge_base_dir"/library/{sources,atoms,views,manifests,indexes,reports}
```

3. Validate local source specs and examples:

```bash
find "$knowledge_base_dir"/sources -maxdepth 2 -name '*.md' -print | sort
```

4. Validate category specs:

```bash
find "$knowledge_base_dir"/entities -maxdepth 1 -name '*.md' -print | sort
```

5. Report status to the operator.

## Not included

- No scheduled job creation.
- No runtime config edit.
- No retrieval index rebuild.
- No external source fetch.
