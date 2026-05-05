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

Allow the operator or agents to drop ad-hoc Markdown notes under `library/sources/manual/` for later extraction.

## Collection contract

- Scan `library/sources/manual/` if it exists.
- Treat each Markdown file as a source record candidate.
- Extract atoms according to `entities/` category definitions.

## Notes

This is useful for arbitrary notes that do not yet have a dedicated source spec.
