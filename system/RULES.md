# Knowledge Library Rules

## Scope

- Static framework artifacts live under `knowledge_base_dir/system/`.
- Mutable local configuration lives under `knowledge_base_dir/workspace/`.
- Generated user knowledge lives under `knowledge_base_dir/data/`.

## Immutability rule

Agents must treat `system/` as read-only unless the operator explicitly asks to change the framework.

## Root indexing rule

Never use `knowledge_base_dir`, `system/`, or `workspace/` as retrieval/search paths. They contain specs, templates, instructions, and local configuration that should not pollute user knowledge retrieval.

## Capture rule

When a task discovers reusable information broader than the direct answer, preserve the broader reusable source and extract atoms.

Example:

- Direct ask: today's school lunch.
- Reusable finding: full monthly menu page.
- Save: source snapshot + dated menu/resource atoms.

## Source rule

Keep source records close to evidence. Source records should include enough text, metadata, and provenance to reconstruct why an atom exists.

## Atom rule

Atoms should be small, answerable, dedupable, and typed. Do not force arbitrary content into topic folders.

## View rule

Views are generated convenience summaries. They must cite atoms/sources and should be safe to regenerate.

## Freshness rule

If information may expire, represent freshness explicitly. Do not silently treat stale knowledge as current.

All atoms require `observed_at`. Mutable facts should also include `valid_until` or `refresh_after` when applicable.

## Correction rule

When a mutable fact is contradicted by newer evidence, follow `system/rules/reconciliation.md`. Never silently overwrite or delete atoms. Corrections are tracked via `corrected_by` / `corrects` chains and inline correction notices in the atom body.

Historical facts (past events with elapsed time ranges) are not corrected unless factually wrong — they transition to `retrieval_status: historical`.

## Deletion/update rule

- Source ingestion behavior: edit `workspace/source-specs/*.md` via runbook.
- Atom kind behavior: edit `system/taxonomy/atom-kinds/*.md` only when explicitly changing framework behavior.
- Runtime/generated knowledge: update under `data/` via runbook.
- Atom corrections: follow `system/rules/reconciliation.md`. Never silently delete or overwrite atoms.
- Do not patch random files for new ingestion requirements.

## Privacy/trust rule

Even in personal deployments, keep fields such as `privacy_tier`, `trust`, and `confidence`. They improve answer quality, provenance, and routing. They do not have to imply multi-user access control.

## No immediate automation rule

Docs may describe scheduled jobs and setup commands. Do not create scheduled jobs or change runtime config unless explicitly requested.
