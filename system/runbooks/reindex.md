# Runbook: Reindex Generated Knowledge

Do not run unless the operator explicitly approves config/index changes.

## Guardrails

- **Role:** retrieval index maintenance runbook.
- **Mutation:** runtime config/index side effects only after explicit approval.
- **Allowed writes:** runtime-specific index storage; no writes to `system/` or `workspace/`.
- **Do not:** index `knowledge_base_dir`, `system/`, or `workspace/`; do not change unrelated runtime config.
- **Before acting:** inspect current retrieval config and show planned path changes.
- **After acting:** verify status/search using runtime-specific commands and report results.
- **Failure mode:** if config tooling or index command is unclear, stop and ask.

## Rule

Index generated folders only. Never index `knowledge_base_dir`, `system/`, or `workspace/`.

## Candidate retrieval paths

```text
knowledge_base_dir/data/sources
knowledge_base_dir/data/atoms
knowledge_base_dir/data/views
knowledge_base_dir/data/reports
```

## Procedure

1. Inspect current runtime retrieval config through the runtime's approved config workflow.
2. Preserve existing config values.
3. Add generated folders to the retrieval/search paths.
4. Apply config only if approved.
5. Rebuild the target retrieval index using the runtime-specific command.

Generic placeholder:

```bash
<agent-runtime> memory index --force
```

6. Verify with runtime-specific status/search commands.

## Notes

If multiple agents should share this library, configure the appropriate shared/default retrieval scope in the local runtime.
