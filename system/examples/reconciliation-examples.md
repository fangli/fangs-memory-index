# Reconciliation Examples

## Example 1: Flight rescheduled

### Day 1 — Initial extraction

Source: conversation where Fang says "I booked a flight to Shanghai on July 1."

Atom written to `data/atoms/2026-05-05.md`:

```markdown
---
id: atom.fang-shanghai-flight-2026q3
kind: event
retrieval_status: active
title: Fang flight to Shanghai
observed_at: 2026-05-05
entities: [fang, shanghai, flight]
time:
  start: 2026-07-01
source_refs: [source.session-2026-05-05-main]
confidence: 0.95
created_at: 2026-05-05T22:00:00-07:00
updated_at: 2026-05-05T22:00:00-07:00
---

Fang booked a flight to Shanghai departing July 1, 2026.
```

SQLite row inserted with `retrieval_status = 'active'`.

### Day 2 — Correction detected

Source: conversation where Fang says "I changed my flight to July 3."

Indexing agent extracts new atom, then queries SQLite for correction candidates:

```sql
SELECT a.* FROM atoms a
JOIN atom_entities ae ON a.id = ae.atom_id
WHERE a.kind = 'event'
  AND a.retrieval_status = 'active'
  AND ae.entity IN ('fang', 'flight', 'shanghai', 'fang li', 'pvg')  -- expanded via entity_aliases
  AND (
    (a.time_start IS NULL AND a.time_end IS NULL)
    OR (a.time_start <= '2026-07-03' AND COALESCE(a.time_end, a.time_start) >= '2026-07-03')
  )
GROUP BY a.id;
```

Finds `atom.fang-shanghai-flight-2026q3`. Detects: same entities, same kind, overlapping time range, different `body_hash`, newer `observed_at`. This is a correction.

**Old atom updated** in `data/atoms/2026-05-05.md`:

```markdown
---
id: atom.fang-shanghai-flight-2026q3
kind: event
retrieval_status: corrected
corrected_by: atom.fang-shanghai-flight-2026q3-v2
corrected_at: 2026-05-06T22:00:00-07:00
title: Fang flight to Shanghai
observed_at: 2026-05-05
entities: [fang, shanghai, flight]
time:
  start: 2026-07-01
source_refs: [source.session-2026-05-05-main]
confidence: 0.95
created_at: 2026-05-05T22:00:00-07:00
updated_at: 2026-05-06T22:00:00-07:00
---

> ⚠️ CORRECTED — This atom has been superseded.
> Current version: atom.fang-shanghai-flight-2026q3-v2 (Fang flight to Shanghai — July 3, in 2026-05-06.md)
> Reason: flight rescheduled

~~Fang booked a flight to Shanghai departing July 1, 2026.~~
```

**New atom written** to `data/atoms/2026-05-06.md`:

```markdown
---
id: atom.fang-shanghai-flight-2026q3-v2
kind: event
retrieval_status: active
title: Fang flight to Shanghai
observed_at: 2026-05-06
corrects: atom.fang-shanghai-flight-2026q3
entities: [fang, shanghai, flight]
time:
  start: 2026-07-03
source_refs: [source.session-2026-05-06-main]
confidence: 0.95
created_at: 2026-05-06T22:00:00-07:00
updated_at: 2026-05-06T22:00:00-07:00
---

Fang flight to Shanghai rescheduled to July 3, 2026 (originally July 1).
```

### Retrieval behavior

User asks: "When is Fang's flight?"

Retrieval agent searches, may find both atoms. Upon reading the old one, sees the correction notice and follows the pointer to `atom.fang-shanghai-flight-2026q3-v2`. Answers: July 3.

User asks: "What was the original flight date?"

Retrieval agent finds the corrected atom (it's still indexed), reads the strikethrough content. Answers: originally July 1, changed to July 3 on May 6.

---

## Example 2: Lunch menu — no conflict

### Month 1

Atom in `data/atoms/2026-04-02.md`:

```markdown
---
id: atom.luna-school-lunch-2026-04
kind: resource
retrieval_status: active
title: Luna school lunch menu — April 2026
observed_at: 2026-04-02
entities: [luna, school, lunch]
freshness:
  valid_from: 2026-04-01
  valid_until: 2026-04-30
source_refs: [source.email-2026-04-02-school]
confidence: 0.9
created_at: 2026-04-02T22:00:00-07:00
updated_at: 2026-04-02T22:00:00-07:00
---

April 2026 lunch menu for Luna's school.
Week 1: Mon pizza, Tue tacos, Wed pasta...
```

### Month 2

New atom in `data/atoms/2026-05-01.md`:

```markdown
---
id: atom.luna-school-lunch-2026-05
kind: resource
retrieval_status: active
title: Luna school lunch menu — May 2026
observed_at: 2026-05-01
entities: [luna, school, lunch]
freshness:
  valid_from: 2026-05-01
  valid_until: 2026-05-31
source_refs: [source.email-2026-05-01-school]
confidence: 0.9
created_at: 2026-05-01T22:00:00-07:00
updated_at: 2026-05-01T22:00:00-07:00
---

May 2026 lunch menu for Luna's school.
Week 1: Mon burgers, Tue chicken...
```

Reconciliation: same entity + kind, but **non-overlapping coverage** (April vs May). No correction. Both remain `active`.

After April 30, the April atom transitions to `retrieval_status: historical` during the next ingestion pass (its `valid_until` has passed). It remains searchable for history queries.

---

## Example 3: Menu revised mid-month

School sends a revised May menu on May 10.

Reconciliation: same entity + kind + **overlapping coverage** (both cover May) + different content + newer `observed_at`. This IS a correction.

Old atom gets `corrected`, new version replaces it. Same flow as the flight example.

---

## Example 3b: Deduplication — same menu re-sent

School re-sends the same May menu email on May 12 (no changes).

Indexing agent extracts atom, computes `body_hash`. Queries:

```sql
SELECT id FROM atoms
WHERE kind = 'resource' AND body_hash = :hash AND retrieval_status = 'active';
```

Finds `atom.luna-school-lunch-2026-05` with matching hash and shared entity `luna`. This is a **duplicate** — skip. Log in `data/manifests/2026-05-12.json`.

---

## Example 4: Preference change

### Day 1

```markdown
---
id: atom.fang-pref-coffee
kind: preference
retrieval_status: active
title: Fang coffee preference
observed_at: 2026-03-15
entities: [fang, coffee]
source_refs: [source.session-2026-03-15]
confidence: 0.8
---

Fang prefers oat milk lattes.
```

### Day 30

Fang says "I've switched to black coffee."

Reconciliation: same entity + kind (preference about fang+coffee), different content, newer `observed_at`. Correction triggered.

Old atom marked corrected. New atom:

```markdown
---
id: atom.fang-pref-coffee-v2
kind: preference
retrieval_status: active
title: Fang coffee preference
observed_at: 2026-04-14
corrects: atom.fang-pref-coffee
entities: [fang, coffee]
source_refs: [source.session-2026-04-14]
confidence: 0.9
---

Fang prefers black coffee (previously oat milk lattes, changed April 2026).
```

---

## Example 5: Correction chain flattening

Fang's flight gets rescheduled three times:

- v1 (May 5): July 1 → `atom.flight-v1`
- v2 (May 6): July 3 → `atom.flight-v2` (corrects v1)
- v3 (May 8): July 5 → `atom.flight-v3` (corrects v2)
- v4 (May 10): July 7 → `atom.flight-v4` (corrects v3)

At v4, the chain depth reaches 4. The indexing agent **flattens**:

- v1: `corrected_by: atom.flight-v4` (was v2, now points directly to latest)
- v2: `corrected_by: atom.flight-v4` (was v3, now points directly to latest)
- v3: `corrected_by: atom.flight-v4`
- v4: `retrieval_status: active`, `corrects: atom.flight-v3`

This ensures the retrieval agent can reach the current version in 1 hop from any prior version.

---

## Example 6: Historical transition — past event

Atom in `data/atoms/2026-04-20.md`:

```markdown
---
id: atom.luna-recital-2026-04
kind: event
retrieval_status: active
title: Luna piano recital
observed_at: 2026-04-20
entities: [luna, piano, recital]
time:
  start: 2026-04-28
source_refs: [source.email-2026-04-20]
confidence: 0.95
---

Luna's piano recital at school, April 28 at 3pm.
```

During May 1 ingestion, the reconciliation pass checks active events:

```sql
SELECT * FROM atoms
WHERE retrieval_status = 'active'
  AND kind = 'event'
  AND time_start < '2026-05-01';
```

Finds `atom.luna-recital-2026-04`. The event has passed (`time_start < today`). No open task references it. Transition to `historical`:

- Update frontmatter: `retrieval_status: historical`, `updated_at: 2026-05-01T...`
- Update SQLite row.
- Do NOT modify the body or add a correction notice.

The atom remains searchable. "What events did Luna have in April?" still finds it.

---

## Example 7: Ambiguous match — logged for review

Day 1: Atom about "Fang meeting with Dr. Chen, June 15."
Day 2: Atom about "Fang appointment with Dr. Chen, June 20."

Same entities (fang, dr. chen), same kind (event), but different dates (June 15 vs June 20) that don't overlap. The indexing agent is uncertain: is this a rescheduled meeting (correction) or two separate appointments?

Since the time ranges don't overlap (both are point events on different days), the candidate query won't match them. Both remain `active`. No ambiguity logged in this case — the time overlap check correctly identifies them as separate events.

But if Day 2 said "Fang meeting with Dr. Chen, June 15 at 3pm" (same date, different detail), the overlap check would match. The agent compares content: if only the time-of-day changed, it's a correction. If the topic/purpose differs significantly, it's ambiguous → log for review.
