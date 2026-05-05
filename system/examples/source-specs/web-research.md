---
source_id: runtime.web_research
status: example
kind: web
owner: operator
collection_mode: opportunistic
frequency: daily
lookback: previous_local_day
trust: primary
outputs:
  - source_record
  - atoms
---

# Web Research Snapshots

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
