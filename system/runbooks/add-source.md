# Runbook: Add, Update, or Delete a Source

## Add a source

1. Read `workspace/AGENTS.md`.
2. Copy `system/examples/source-spec-template.md` or an example from `system/examples/source-specs/` to `workspace/source-specs/<source-name>.md`.
3. Fill frontmatter:
   - `source_id`
   - `status`
   - `kind`
   - `collection_mode`
   - `frequency`
   - `lookback`
   - `trust`
   - `outputs`
4. Describe collection contract, extraction hints, freshness, and output requirements.
5. Do not edit daily ingestion logic unless the new source requires a genuinely new collector capability.

## Update a source

Edit the existing local spec file. Preserve `source_id` unless this is a new logical source.

## Disable/delete a source

Prefer `status: disabled` when historical records should remain understandable. Delete only if the operator explicitly wants the source removed from future specs.

## Acceptance check

After source changes, a future dry run should be able to list enabled source specs and explain what each one collects.
