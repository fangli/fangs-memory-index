---
source_id: local.manual_inbox
status: example
kind: manual
owner: operator
collection_mode: manual
frequency: on_demand
trust: mixed
outputs:
  - source_record
  - atoms
---

# Manual Inbox

## Purpose

Allow the operator or agents to drop ad-hoc Markdown notes under `data/sources/manual/` for later extraction.

## Collection contract

- Scan `data/sources/manual/` if it exists.
- Treat each Markdown file as a source record candidate.
- Extract atoms according to `system/taxonomy/atom-kinds/` category definitions.

## Notes

This is useful for arbitrary notes that do not yet have a dedicated source spec.
