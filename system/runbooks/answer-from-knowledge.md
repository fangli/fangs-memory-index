# Runbook: Answer From Knowledge

Purpose: decide when to answer from local knowledge and when to refresh/research.

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
   - If missing or stale, research/fetch and preserve the broader reusable source.
5. Answer concisely and include provenance when it affects trust.

## Example: agenda this week

- Query event/task atoms overlapping this week.
- Include live calendar data if available/requested.
- Include source-derived agenda from messages/chats/files.
- Exclude stale or completed tasks unless relevant.

## Example: recurring schedule/menu lookup

- Search for resource atoms covering the requested entity/date.
- If found and fresh, answer from memory.
- If missing, fetch current source, answer, then save full schedule as source/resource atoms.
