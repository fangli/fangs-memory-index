---
source_id: example.source
status: draft
kind: custom
owner: operator
collection_mode: passive
frequency: daily
lookback: previous_local_day
trust: mixed
outputs:
  - source_record
  - atoms
---

# Example Source Spec

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
