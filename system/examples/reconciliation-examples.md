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

Indexing agent extracts new atom, then queries SQLite:

```sql
SELECT * FROM atoms
WHERE kind = 'event'
  AND retrieval_status = 'active'
  AND id IN (SELECT atom_id FROM atom_entities WHERE entity IN ('fang', 'flight', 'shanghai'))
  AND time_start BETWEEN '2026-06-15' AND '2026-07-15';
```

Finds `atom.fang-shanghai-flight-2026q3`. Detects: same entities, same kind, nearby time, different `time_start`, newer `observed_at`. This is a correction.

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
