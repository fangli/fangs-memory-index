# Reconciliation Rules

During daily ingestion, after extracting new atoms, the indexing agent runs a reconciliation pass to detect and handle corrections, expiry, and staleness.

## Correction Detection

A new atom N is a **correction** of existing atom M when ALL of:

1. Same `kind`
2. At least one shared entity (≥1 overlap in entity sets)
3. Overlapping or identical time range (or both timeless for that kind)
4. Different substantive content (title, time fields, statement, or status differ)
5. N has a later `observed_at` than M
6. M has `retrieval_status: active` (not already corrected)

When detected:

- M gets: `retrieval_status: corrected`, `corrected_by: N.id`, `corrected_at: now`
- N gets: `retrieval_status: active`, `corrects: M.id`
- The Markdown file for M is updated with a correction notice (see Atom Correction Format below)
- N is written normally

### Identical content (deduplication)

If N has the same substantive content as M (same title, time, status, statement after normalization), skip N as a duplicate. Record the skip in `data/manifests/`.

## Correction Policies by Kind

| Kind | Match criteria | Resolution |
|------|---------------|-----------|
| event | Same entity + overlapping time | Newer `observed_at` wins |
| task | Same entity + same title/topic | Newer wins; `done` supersedes `open` |
| claim | Same entity + same field/topic | Newer wins if confidence gap > 0.2 or later `observed_at` |
| preference | Same subject | Newer always wins (people change minds) |
| decision | Same topic/scope | Newer always wins; keep chain |
| resource | Same entity + same `resource_type` + overlapping coverage | Newer wins |
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

## Expiry

Atoms with `valid_until < today` that have `retrieval_status: active`:

- If the atom is a forward-looking mutable fact (future event that has now passed without being observed as completed): transition to `retrieval_status: historical`
- Do NOT mark as corrected — expiry is not a correction, it's a natural lifecycle transition

Historical atoms remain searchable. "What was Luna's lunch last December?" should still work.

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

## Ambiguous Matches

If the indexing agent is uncertain whether N truly corrects M (low entity overlap, unclear time relationship, or different enough to be separate facts):

- Do NOT auto-correct
- Write both as `active`
- Log the ambiguity in the daily report under "Needs review"
- A human or future ingestion pass can resolve it
