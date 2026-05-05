# Installation Runbook

This file describes how another agent should initialize the knowledge library later. It is intentionally not executed by merely creating or cloning this repository.

## Prerequisites

- An AI-agent runtime that can read/write files.
- Optional: a scheduler/cron-like system.
- Optional: a semantic retrieval index.
- Access to `knowledge_base_dir`.
- Operator approval before making config or scheduler changes.

## Step 1 — Choose `knowledge_base_dir`

Clone or copy this project to the desired base directory.

Example:

```bash
cd /path/to/knowledge_base_dir
find . -maxdepth 3 -type f | sort
```

Expected top-level files include `AGENTS.md`, `README.md`, `LICENSE`, and `CHANGELOG.md`. Static framework docs live under `system/`.

## Step 2 — Initialize generated folders

Generated folders are specified as:

```text
data/sources
data/atoms
data/views
data/manifests
data/indexes
data/reports
```

If missing, create them under `knowledge_base_dir`.

## Step 3 — Add local source specs

Source specs are local configuration and are ignored by default unless explicitly force-added.

```bash
cp system/examples/source-specs/agent-session-logs.md workspace/source-specs/my-agent-session-logs.md
```

Edit the copied file to match the local runtime's transcript paths, time zone, and collection rules.

## Step 4 — Configure retrieval paths

Do not index `knowledge_base_dir`, `system/`, or `workspace/`.

When the operator asks to enable indexing, add only generated Markdown folders to the runtime's retrieval/search configuration, for example:

```text
knowledge_base_dir/data/sources
knowledge_base_dir/data/atoms
knowledge_base_dir/data/views
knowledge_base_dir/data/reports
```

Preserve existing runtime configuration values.

## Step 5 — Build or refresh the retrieval index

Only after retrieval paths are configured and the operator approves, run the runtime-specific index rebuild command.

Generic placeholder:

```bash
<agent-runtime> memory index --force
```

Replace with the actual command for the local runtime.

## Step 6 — Create daily librarian schedule

Only after operator approval, create a scheduler job similar to:

```text
name: knowledge-librarian-daily
schedule: 03:50 local time daily
payload: run the daily ingestion runbook for previous local calendar day
execution: isolated/non-interactive agent task if available
```

The job prompt should point the agent to:

- `knowledge_base_dir/AGENTS.md`
- `knowledge_base_dir/system/runbooks/daily-ingestion.md`

## Step 7 — First manual dry run

Before enabling recurrence, run the daily ingestion runbook manually for one date with `dry_run: true`, inspect outputs, then run once for real if approved.

## Step 8 — Maintenance

- Add new local source specs under `workspace/source-specs/`.
- Update framework/category behavior under `system/` only when explicitly changing the framework.
- Keep generated output under `data/`.
- Keep this installation doc current as the system evolves.
