# Read-Only Framework Boundary

Files under `system/` are static framework files.

Agents should read them to understand how the memory index works, but should not modify them during normal ingestion, source collection, indexing, or answering workflows.

Allowed writes by default:

```text
workspace/  local configuration via runbook
data/       generated user knowledge via runbook
```

Modify `system/` only when the operator explicitly asks to change the framework, schemas, taxonomy, templates, examples, or runbooks.
