# Reconciliation Rules

During daily ingestion, after extracting new atoms, the indexing agent runs a reconciliation pass to detect and handle corrections, expiry, and staleness.

## Entity Normalization for Matching

Before comparing entity sets between atoms, normalize each entity:

1. Lowercase.
2. Strip leading/trailing whitespace.
3. Collapse internal whitespace to a single space.
4. Expand through `entity_aliases` table: if entity E has aliases, include all aliases and the canonical form in the comparison set.

Two atoms share an entity when their expanded entity sets have at least one common value.

## Correction Detection

A new atom N is a **correction** of existing atom M when ALL of:

1. Same `kind`
2. At least one shared entity (≥1 overlap in expanded entity sets — see normalization above)
3. Overlapping or identical time range (or both timeless for that kind)
4. Different substantive content (title, time fields, statement, or status differ) — equivalently, different `body_hash`
5. N has a later `observed_at` than M
6. M has `retrieval_status: active` (not already corrected)

### Time overlap check

Two atoms have overlapping time when:

```
N.time_start <= M.time_end_effective AND N.time_end_effective >= M.time_start
```

Where `time_end_effective` = `time_end` if set, otherwise `time_start` (point event). If both atoms have no time fields (timeless), they overlap by definition for that kind.

### Candidate query

For events and tasks, use `time_start`/`time_end` for overlap:

```sql
SELECT a.* FROM atoms a
JOIN atom_entities ae ON a.id = ae.atom_id
WHERE a.kind = :new_kind
  AND a.retrieval_status = 'active'
  AND ae.entity IN (:expanded_entities)
  AND (
    (a.time_start IS NULL AND a.time_end IS NULL)
    OR (a.time_start <= :new_time_end_effective AND
        COALESCE(a.time_end, a.time_start) >= :new_time_start)
  )
GROUP BY a.id;
```

For resources, use `valid_from`/`valid_until` for coverage overlap:

```sql
SELECT a.* FROM atoms a
JOIN atom_entities ae ON a.id = ae.atom_id
WHERE a.kind = 'resource'
  AND a.retrieval_status = 'active'
  AND a.resource_type = :new_resource_type
  AND ae.entity IN (:expanded_entities)
  AND (
    (a.valid_from IS NULL AND a.valid_until IS NULL)
    OR (a.valid_from <= COALESCE(:new_valid_until, :new_valid_from) AND
        COALESCE(a.valid_until, a.valid_from) >= :new_valid_from)
  )
GROUP BY a.id;
```

For timeless kinds (preference, decision, procedure, claim), omit time/coverage filters — match by entity + kind only:

```sql
SELECT a.* FROM atoms a
JOIN atom_entities ae ON a.id = ae.atom_id
WHERE a.kind = :new_kind
  AND a.retrieval_status = 'active'
  AND ae.entity IN (:expanded_entities)
GROUP BY a.id;
```

Then filter all candidate results by `body_hash != :new_body_hash` and `observed_at < :new_observed_at`.

When detected:

- M gets: `retrieval_status: corrected`, `corrected_by: N.id`, `corrected_at: now`
- N gets: `retrieval_status: active`, `corrects: M.id`
- The Markdown file for M is updated with a correction notice (see Atom Correction Format below)
- N is written normally

### Identical content (deduplication)

If N has the same `body_hash` as M (same kind, shared entity, same substantive content), skip N as a duplicate. Record the skip in `data/manifests/`.

## Correction Policies by Kind

| Kind | Match criteria | Resolution |
|------|---------------|-----------|
| event | Same entity + overlapping time | Newer `observed_at` wins |
| task | Same entity + same title/topic | Newer wins; `done` supersedes `open` |
| claim | Same entity + same field/topic | Newer wins if confidence gap > 0.2 or later `observed_at` |
| preference | Same subject | Newer always wins (people change minds) |
| decision | Same topic/scope | Newer always wins; keep chain |
| resource | Same entity + same `resource_type` + overlapping coverage (`valid_from`/`valid_until`) | Newer wins |
| procedure | Same title/scope | Newer wins |
| observation | — | Observations do not conflict; they coexist |

## Atom Correction Format

When an atom is corrected, its Markdown entry is updated to include a correction notice **above** the original content:

```markdown
---
id: atom.flight-july-2026
kind: event
retrieval_status: corrected
corrected_by: atom.flight-july-2026-v2
corrected_at: 2026-05-06T10:00:00-07:00
---

> ⚠️ CORRECTED — This atom has been superseded.
> Current version: atom.flight-july-2026-v2 (Fang flight July 3, in 2026-05-06.md)
> Reason: flight rescheduled

~~Fang flight to Shanghai — July 1, 2026~~

Departure July 1, 2026.
```

The retrieval agent, upon finding this atom, will see the correction notice and follow the pointer to the current version.

## Expiry and Historical Transitions

Atoms transition to `retrieval_status: historical` based on kind-specific criteria. This is NOT a correction — it's a natural lifecycle transition. Historical atoms remain searchable for history queries.

### Transition criteria by kind

| Kind | Transition to `historical` when |
|------|----------------------------------|
| event | `time_start < today` (event has passed) and no open follow-up task references it |
| task | `status = done`, OR (`time_due < today` AND status is not `open` or `blocked`) |
| resource | `valid_until < today` |
| claim | Never auto-expires; only via explicit correction |
| preference | Never auto-expires; only via explicit correction |
| decision | Never auto-expires; only via explicit correction |
| procedure | Never auto-expires; only via explicit correction |
| observation | Never auto-expires; observations are inherently point-in-time |

### Rules

- Only transition atoms with `retrieval_status: active`.
- Do NOT mark as corrected — expiry is not a correction.
- Do NOT delete or modify the atom body. Only change `retrieval_status` and `updated_at` in frontmatter.
- Update the SQLite index row accordingly.
- Tasks with `status: open` or `status: blocked` that are past due do NOT auto-transition — they remain active (they represent unfinished work).

## Staleness (refresh_after)

Atoms past `refresh_after` but without contradiction:

- Do NOT change `retrieval_status` — the fact might still be true
- Add to `data/views/refresh-needed.md` so the system knows to re-research when relevant
- Only correct when new contradicting evidence actually arrives

## Multiple Corrections

If a fact is corrected multiple times (flight: July 1 → July 3 → July 5):

- v1: `corrected_by: v2`
- v2: `corrected_by: v3`
- v3: `retrieval_status: active`

Each intermediate version points forward. The retrieval agent can follow the chain to reach the current version from any entry point.

### Chain depth limit

The retrieval agent follows correction pointers up to **5 hops**. If a chain exceeds 5, the agent reports the anomaly and uses the last reachable version. The indexing agent should avoid chains deeper than 5 by pointing all prior versions directly to the latest when a 4th correction occurs (flatten the chain).

## Ambiguous Matches

If the indexing agent is uncertain whether N truly corrects M (low entity overlap, unclear time relationship, or different enough to be separate facts):

- Do NOT auto-correct
- Write both as `active`
- Log the ambiguity in the daily report under "Needs review"
- A human or future ingestion pass can resolve it
