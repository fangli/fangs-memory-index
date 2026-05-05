---
source_id: local.manual_inbox
status: example
kind: manual
owner: operator
collection_mode: manual
frequency: on_demand
trust: mixed
allowed_actions:
  - read_local_files
writes_allowed:
  - data/sources
  - data/atoms
dedupe_key:
  - source_id
  - origin.id
  - hash
freshness_policy:
  default: unknown
outputs:
  - source_record
  - atoms
---

# Manual Inbox

## Guardrails

- **Role:** example source spec.
- **Mutation:** read-only example; copy to `workspace/source-specs/` before local edits.
- **Allowed writes:** examples authorize no writes until copied into a local spec.
- **Do not:** treat `status: example` as enabled ingestion.
- **Before acting:** verify a local copied spec exists with `status: enabled`.
- **Failure mode:** if only the example exists, skip ingestion and report that no local source spec is enabled.

## Purpose

Allow the operator or agents to drop ad-hoc Markdown notes under `data/sources/manual/` for later extraction.

## Collection contract

- Scan `data/sources/manual/` if it exists.
- Treat each Markdown file as a source record candidate.
- Extract atoms according to `system/taxonomy/atom-kinds/` category definitions.

## Notes

This is useful for arbitrary notes that do not yet have a dedicated source spec.
