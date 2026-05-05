# Runbook: Add, Update, or Delete a Source

## Guardrails

- **Role:** local source configuration runbook.
- **Mutation:** local config only.
- **Allowed writes:** `workspace/source-specs/*.md`.
- **Do not:** edit `system/examples/` when adding a local source; do not write generated records to `workspace/`.
- **Before acting:** decide whether the request is a local source config change or a framework/example change.
- **After acting:** report the source spec path, status, collection mode, allowed actions, and any missing capabilities.
- **Failure mode:** if the source requires new collector code or unclear credentials, create the spec as `status: draft` and report the blocker.

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
   - `allowed_actions`
   - `writes_allowed`
   - `dedupe_key`
4. Describe collection contract, extraction hints, freshness, and output requirements.
5. Do not edit daily ingestion logic unless the new source requires a genuinely new collector capability.

## Update a source

Edit the existing local spec file. Preserve `source_id` unless this is a new logical source.

## Disable/delete a source

Prefer `status: disabled` when historical records should remain understandable. Delete only if the operator explicitly wants the source removed from future specs.

## Acceptance check

After source changes, a future dry run should be able to list enabled source specs and explain what each one collects.
