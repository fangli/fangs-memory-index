# Guardrails

These guardrails are part of the static framework. Agents must follow them before executing any runbook or source spec.

## Instruction priority inside this repository

When repository instructions conflict, apply this order:

1. The operator's explicit current request.
2. Root `AGENTS.md` boundary rules.
3. `system/GUARDRAILS.md` and `system/RULES.md`.
4. The specific runbook being executed.
5. Local source specs under `workspace/source-specs/`.
6. Examples/templates.

If conflict remains, stop and report the conflict. Do not guess.

## Zone mutation rules

```text
system/     read-only unless explicitly changing framework
workspace/  editable local configuration via runbook
data/       generated user knowledge via runbook
```

Agents must resolve write paths before writing. If a write target is outside the allowed zone for the current task, stop.

## Side-effect rules

Reading docs or runbooks must not create side effects.

Do not do any of the following unless the operator explicitly requested it in the current task:

- create, update, or delete scheduled jobs;
- edit runtime configuration;
- rebuild retrieval indexes;
- fetch network/external sources;
- delete generated user data;
- modify static framework files under `system/`.

## Dry-run rule

For first-time ingestion, changed source specs, changed schemas, or new collectors:

1. perform a dry run when possible;
2. report planned writes and skipped records;
3. write only after approval or when the runbook explicitly says the current task is allowed to write.

## Provenance rule

Every atom must cite at least one `source_ref`. If no source record exists, create one first or mark extraction incomplete. Do not create orphan atoms.

## Freshness rule

Mutable facts must include freshness metadata: `observed_at` (required on all atoms) plus at least one of `valid_until`, `refresh_after`, or a note that freshness is unknown.

If freshness is unknown, mark it as unknown. Do not imply stale information is current.

## Correction rule

When new evidence contradicts an existing active atom, the indexing agent must follow `system/rules/reconciliation.md`:

1. Mark the old atom `retrieval_status: corrected` with a `corrected_by` pointer.
2. Add an inline correction notice to the old atom's Markdown body.
3. Write the new atom with `corrects` pointing back.
4. Update `data/indexes/knowledge.sqlite`.

Do not delete corrected atoms. They remain in indexed directories for history/audit queries. The retrieval agent reads the correction notice and follows the pointer to the current version.

## Dedupe rule

Before writing new source records or atoms, check existing records or manifests for matching source IDs, content hashes, normalized titles, times, and entities.

## Reconciliation rule

During daily ingestion, after atom extraction, run the reconciliation pass defined in `system/rules/reconciliation.md`. This detects corrections, transitions expired atoms to `historical`, and flags ambiguous matches.

Do not silently overwrite or delete atoms. Corrections are tracked via `corrected_by` / `corrects` chains and inline correction notices.

## Retrieval/indexing rule

Never configure retrieval/indexing over:

- `knowledge_base_dir`
- `system/`
- `workspace/`

Only selected generated subfolders under `data/` may be indexed.

## Failure behavior

If required tools, paths, source specs, credentials, or permissions are missing:

1. stop the operation;
2. report what is missing;
3. do not substitute unrelated sources or invent data.
