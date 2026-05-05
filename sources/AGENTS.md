# sources/AGENTS.md — Pluggable Source Registry

This folder defines what data sources the daily librarian should inspect.

Local source specs are intentionally gitignored by default because they often contain user-specific paths, account names, or workflow details.

To add a new source locally, copy a file from `sources/examples/` or `sources/TEMPLATE.md` into `sources/<name>.md`, then edit it. To disable a source, edit `status: disabled` or delete the local spec file. Do not hard-code source behavior elsewhere unless the source spec explicitly points to a helper.

## Daily librarian behavior

1. Read this file.
2. Read all local `*.md` files in this folder except `AGENTS.md` and `TEMPLATE.md`.
3. Optionally read `examples/*.md` for guidance, but do not treat examples as enabled local sources unless copied into this folder.
4. For each spec with `status: enabled`, follow its collection contract.
5. Write normalized source records under `../library/sources/`.
6. Extract atoms under `../library/atoms/`.
7. Record results under `../library/manifests/` and `../library/reports/`.

## Source spec frontmatter

```yaml
source_id: stable.source.id
status: enabled | disabled | draft
kind: session_logs | message | chat | files | web | manual | calendar | custom
owner: operator | team | system
collection_mode: passive | active | opportunistic | manual
frequency: daily | on_demand | opportunistic
lookback: previous_local_day
trust: primary | secondary | assistant-derived | mixed
outputs:
  - source_record
  - atoms
  - view_updates
```

## Naming

Use stable kebab-case filenames:

```text
agent-session-logs.md
processed-messages.md
conversation-files.md
web-research-snapshots.md
manual-inbox.md
```
