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

1. **Semantic retrieval** over generated Markdown under `data/`. The retrieval agent reads full atom files and follows inline correction notices (e.g., "CORRECTED — see current version at..."). When the retrieval system supports metadata filtering, prefer filtering `retrieval_status = 'active'` at query time to avoid surfacing corrected atoms.
2. **Deterministic query layer** backed by `data/indexes/knowledge.sqlite` — used by the indexing agent for reconciliation, view generation, and structured queries.

The retrieval agent does not query SQLite directly. It relies on Markdown content and inline instructions within atoms. However, views generated from SQLite (agenda, open loops) pre-filter to active atoms, providing an indirect benefit.

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

- `observed_at` (required)
- `valid_from`
- `valid_until`
- `refresh_after`

If freshness is unknown, store the information anyway but mark it as `unknown` instead of pretending it is current.

## Correction model

Mutable facts (forward-looking events, active plans, current statuses) are subject to correction when new contradicting evidence arrives. Corrected atoms remain in the same files but are marked with `retrieval_status: corrected` and an inline notice pointing to the replacement.

Historical facts (past events, consumed resources) are never corrected unless factually wrong. They transition to `retrieval_status: historical` once their time range passes.

Key mechanisms:

- **Detection:** same kind + shared entity (via alias expansion) + overlapping time + different `body_hash` + newer `observed_at`.
- **Resolution:** old atom marked `corrected` with inline notice; new atom links back via `corrects`.
- **Chain flattening:** when a 4th correction occurs, all prior versions point directly to the latest (avoids deep chains).
- **Retrieval safety:** corrected atoms contain a visible `⚠️ CORRECTED` notice; retrieval agents follow the pointer. Views only include `active` atoms.

See `system/rules/reconciliation.md` for full correction detection and resolution policies.

## Pluggability

Source ingestion is controlled by Markdown specs in `workspace/source-specs/`.
Adding a source means adding one local spec file. Removing a source means deleting or disabling its local spec file.

The daily librarian should read all enabled local source specs and execute their collection instructions.
