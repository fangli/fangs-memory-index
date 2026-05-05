---
source_id: runtime.web_research
status: example
kind: web
owner: operator
collection_mode: opportunistic
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
---

# Web Research Snapshots

## Guardrails

- **Role:** example source spec.
- **Mutation:** read-only example; copy to `workspace/source-specs/` before local edits.
- **Allowed writes:** examples authorize no writes until copied into a local spec.
- **Do not:** treat `status: example` as enabled ingestion.
- **Before acting:** verify a local copied spec exists with `status: enabled`.
- **Failure mode:** if only the example exists, skip ingestion and report that no local source spec is enabled.

## Purpose

Preserve reusable web research results discovered while answering questions.

## Collection contract

- Detect URLs and web-fetch/browser results in sessions.
- Save page title, URL, fetched_at, quoted/summarized content, and coverage if known.
- If the assistant only answered from a URL without enough source text, create a low-confidence source record and mark `needs_refresh: true`.

## Extraction hints

- Schedules/menus/calendars -> `resource` plus dated `event`/`claim` atoms.
- Policy pages -> `resource` plus claims with `refresh_after`.
- Current mutable facts must retain `fetched_at`.

## Freshness defaults

- Monthly schedule/menu: refresh after coverage end.
- Policy pages: refresh after 30 days unless source says otherwise.
- News/current facts: refresh after 7 days unless clearly durable.
