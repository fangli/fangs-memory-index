# Personal Memory Index Specification

## Problem

AI agents often answer a direct question after doing useful research, but only the final answer survives in conversation history. The broader reusable source may be lost.

Example: if an agent researches today's school lunch, it should also preserve the broader lunch schedule it found so future questions can answer from local knowledge instead of repeating research.

The domain is intentionally unbounded: personal life, work, family, files, web policies, opinions, agendas, tasks, one-off facts, durable facts, expiring facts, and fuzzy time ranges.

## Design principle

Do not make topic folders. Topics are metadata.

Store information by behavioral kind:

- source
- atom
- view
- taxonomy/category definition
- ingestion source spec

## Repository zones

```text
system/     static framework files; read-only by default
workspace/  mutable local configuration, especially source specs
data/       generated user knowledge and indexes
```

## Object model

### Source

A normalized record of evidence or input.

Examples:

- message or email-like item
- chat turn
- attached document text
- web page snapshot
- calendar export
- manually supplied note
- assistant research output

### Atom

A compact reusable unit extracted from one or more sources.

Canonical atom kinds:

- `event`
- `task`
- `claim`
- `preference`
- `decision`
- `resource`
- `procedure`
- `observation`

### View

A generated rollup for common query shapes.

Examples:

- agenda next 14 days
- open loops
- recently changed facts
- expiring resources
- entity digest

Views are not authoritative evidence. They cite atoms/sources.

## Retrieval strategy

Use two complementary retrieval surfaces:

1. **Semantic retrieval** over generated Markdown under `data/`.
2. **Deterministic query layer** for time/entity/status filters, eventually backed by `data/indexes/knowledge.sqlite` or an equivalent local database.

This supports questions like:

```text
what agenda do I have this week?
what does my child need for school today?
what are my open loops?
what do we know about this source/page/policy?
```

## Fuzzy agenda interpretation

For agenda-like questions:

1. Interpret fuzzy time in the operator's configured local timezone.
2. Query event/task atoms by overlap with the date range.
3. Include resources only if they have date coverage or explicit agenda relevance.
4. Use semantic retrieval to expand entities/topics.
5. Prefer atoms with primary source evidence and recent freshness.
6. Answer with source/provenance when useful.

## Freshness

Every atom can have:

- `valid_from`
- `valid_until`
- `observed_at`
- `refresh_after`
- `staleness_policy`

If freshness is unknown, store the information anyway but mark it as `unknown` instead of pretending it is current.

## Pluggability

Source ingestion is controlled by Markdown specs in `workspace/source-specs/`.
Adding a source means adding one local spec file. Removing a source means deleting or disabling its local spec file.

The daily librarian should read all enabled local source specs and execute their collection instructions.
