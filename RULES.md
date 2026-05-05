# Knowledge Library Rules

## Scope

All project artifacts live under `knowledge_base_dir`.
Generated user knowledge lives under `knowledge_base_dir/library/`.

## Root indexing rule

Never use `knowledge_base_dir` itself as a retrieval/search path. It contains specs, templates, and instructions that should not pollute user knowledge retrieval.

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

## Deletion/update rule

- Source ingestion behavior: edit `sources/*.md`.
- Atom kind behavior: edit `entities/*.md`.
- Runtime/generated knowledge: update under `library/` via runbook.
- Do not patch random files for new ingestion requirements.

## Privacy/trust rule

Even in personal deployments, keep fields such as `privacy_tier`, `trust`, and `confidence`. They improve answer quality, provenance, and routing. They do not have to imply multi-user access control.

## No immediate automation rule

Docs may describe scheduled jobs and setup commands. Do not create scheduled jobs or change runtime config unless explicitly requested.
