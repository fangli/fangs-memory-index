# Runbook: Answer From Knowledge

Purpose: decide when to answer from local knowledge and when to refresh/research.

## Guardrails

- **Role:** retrieval/answering runbook.
- **Mutation:** read-only by default.
- **Allowed writes:** none, unless the operator explicitly asks to capture newly researched reusable sources.
- **Do not:** write atoms, fetch external sources, rebuild indexes, or update source specs merely because an answer is missing.
- **Before acting:** check generated knowledge first when retrieval is installed; check freshness before trusting mutable facts.
- **After acting:** cite provenance when it affects trust; state if knowledge was missing or stale.
- **Failure mode:** if available knowledge is insufficient, say what needs refresh/capture instead of inventing an answer.

## Steps

1. Interpret the user's question shape:
   - agenda/time range
   - open loops/tasks
   - factual lookup
   - resource reuse
   - preference/decision recall
   - current mutable fact
2. Search generated knowledge paths if installed:
   - atoms for precise reusable facts
   - views for summaries
   - sources for provenance
3. For fuzzy time questions, normalize range in the operator's configured local timezone.
4. Check freshness:
   - If atom/source covers the requested date/range and is not stale, answer from knowledge.
   - If missing or stale, research/fetch only when the operator's task permits it, then preserve the broader reusable source if capture is allowed.
5. Answer concisely and include provenance when it affects trust.

## Example: agenda this week

- Query event/task atoms overlapping this week.
- Include live calendar data if available/requested.
- Include source-derived agenda from messages/chats/files.
- Exclude stale or completed tasks unless relevant.

## Example: recurring schedule/menu lookup

- Search for resource atoms covering the requested entity/date.
- If found and fresh, answer from memory.
- If missing and research is allowed, fetch current source, answer, then save full schedule as source/resource atoms if capture is allowed.
