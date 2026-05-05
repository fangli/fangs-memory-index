---
source_id: example.source
status: draft
kind: custom
owner: operator
collection_mode: passive
frequency: daily
lookback: previous_local_day
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

# Example Source Spec

## Guardrails

- **Role:** local source spec template.
- **Mutation:** copy into `workspace/source-specs/` before editing for a local installation.
- **Allowed writes:** copied local specs may authorize writes only under `data/`.
- **Do not:** put credentials, secrets, or destructive shell commands in source specs.
- **Before acting:** validate `allowed_actions`, `writes_allowed`, and `dedupe_key`.
- **Failure mode:** if required source access is unclear, keep status as `draft`.

## Purpose

Describe what this source contributes and why it is useful.

## Collection contract

- Where to look:
- Time range:
- How to identify already-seen records:
- What to ignore:
- What to save as source records:

## Extraction hints

- Event patterns:
- Task patterns:
- Resource patterns:
- Claim/preference/decision patterns:

## Freshness

Describe expiry/refresh behavior.

## Output requirements

Generated source records go to `data/sources/`.
Generated atoms go to `data/atoms/`.
