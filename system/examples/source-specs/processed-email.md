---
source_id: local.processed_messages
status: example
kind: message
owner: operator
collection_mode: active
frequency: daily
lookback: previous_local_day
trust: primary
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
  - view_updates
---

# Processed Messages

## Guardrails

- **Role:** example source spec.
- **Mutation:** read-only example; copy to `workspace/source-specs/` before local edits.
- **Allowed writes:** examples authorize no writes until copied into a local spec.
- **Do not:** treat `status: example` as enabled ingestion.
- **Before acting:** verify a local copied spec exists with `status: enabled`.
- **Failure mode:** if only the example exists, skip ingestion and report that no local source spec is enabled.

## Purpose

Capture useful facts, events, tasks, notices, resources, and agenda items from email-like or inbox-like messages processed by the agent.

## Collection contract

Runtime-specific inbox integration is not installed by this example. When enabled, the collector should use stable message IDs and avoid reprocessing via `data/manifests/seen-hashes.json` or an equivalent manifest.

## Extraction hints

- Dates, date ranges, RSVP deadlines, pickup/dropoff changes -> `event` or `task`.
- Monthly menus, policy pages, newsletters, flyers -> `resource` plus any dated atoms.
- Personal durable context -> `claim` or `preference`.

## Freshness

Most message source records do not refresh; extracted atoms may expire based on dates in the message.
