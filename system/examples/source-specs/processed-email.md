---
source_id: local.processed_messages
status: example
kind: message
owner: operator
collection_mode: active
frequency: daily
lookback: previous_local_day
trust: primary
outputs:
  - source_record
  - atoms
  - view_updates
---

# Processed Messages

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
