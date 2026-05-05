# Runbook: Daily Knowledge Ingestion

Purpose: process the previous local day into source records, atoms, and views.

This is the runbook a future scheduled job should execute. It is not a scheduled job by itself.

## Guardrails

- **Role:** generated-data ingestion runbook.
- **Mutation:** generated-only.
- **Allowed writes:** `data/sources/`, `data/atoms/`, `data/views/`, `data/manifests/`, `data/reports/`; `data/indexes/` only if the runbook/tool explicitly maintains a local deterministic index.
- **Do not:** modify `system/`, modify `workspace/`, create scheduled jobs, edit runtime config, or rebuild retrieval indexes unless explicitly requested.
- **Before acting:** resolve `knowledge_base_dir`, read `system/GUARDRAILS.md`, and validate enabled local source specs.
- **After acting:** write/report source count, atom count, duplicates skipped, files written, and blockers.
- **Failure mode:** if a source spec is unsafe, incomplete, or requires unavailable tools, skip it and report; do not improvise.

## Inputs

- Local timezone: operator-configured timezone.
- Date range: previous local calendar day unless specified otherwise.
- Source specs: all enabled local `workspace/source-specs/*.md` specs.
- Taxonomy specs: all `system/taxonomy/atom-kinds/*.md` specs.

## Reading Order (Entry Point for Indexing Agent)

The indexing agent must read these files in order before beginning work. This is the canonical entry point — no other reading sequence is valid.

### Phase 1: Understand the system

1. This file (`system/runbooks/daily-ingestion.md`) — you are here.
2. `AGENTS.md` (repository root) — boundary guide.
3. `system/GUARDRAILS.md` — immutable safety rules.
4. `system/RULES.md` — operational rules.
5. `system/SPEC.md` — architecture and retrieval strategy.

### Phase 2: Understand the data model

6. `system/schemas/atom-record.md` — atom structure, required/optional fields, correction format.
7. `system/schemas/source-record.md` — source structure.
8. `system/schemas/index-schema.md` — SQLite schema, entity normalization, `body_hash` computation, rebuild procedure.
9. `system/schemas/view-record.md` — view structure.

### Phase 3: Understand reconciliation

10. `system/rules/reconciliation.md` — correction detection, time overlap, entity matching, expiry transitions, chain flattening, ambiguity handling.

### Phase 4: Understand what to extract

11. All files in `system/taxonomy/atom-kinds/` — one per kind. Read every `.md` file in that directory.

### Phase 5: Understand what to collect

12. `workspace/AGENTS.md` — local configuration guide.
13. All enabled `workspace/source-specs/*.md` — one per source. Skip files with `status: disabled` or `status: draft`.

### Phase 6: Check examples (optional, first run only)

14. `system/examples/reconciliation-examples.md` — concrete correction/dedup/expiry scenarios.
15. `system/examples/source-specs/` — example source specs for reference.

After completing this reading order, proceed to the Workflow below.

## Workflow

### Preconditions

0. **Ensure SQLite index exists.** If `data/indexes/knowledge.sqlite` does not exist, create it using the full schema from `system/schemas/index-schema.md` before proceeding. This is required for reconciliation and dedup queries.

### Collection (steps 1–4)

1. Determine the target date range (default: previous local calendar day).
2. For each enabled source spec:
   - Read the spec's collection contract.
   - Collect candidate records for the date range using the spec's `allowed_actions`.
   - Compute content hash for each candidate.
   - Check `data/manifests/` for existing hashes — skip duplicates.
   - Write normalized source records under `data/sources/YYYY-MM-DD.md` (one file per day, multiple records per file).
3. For each source record written:
   - Extract atoms based on `system/taxonomy/atom-kinds/` classification rules.
   - Assign `id`, `kind`, `entities`, `tags`, `time`, `freshness`, `confidence`, `observed_at`, `source_refs`.
   - Compute `body_hash` (SHA-256 of normalized body content).
   - Write atoms to `data/atoms/YYYY-MM-DD.md` (one file per day, multiple atoms per file).
4. Record all source hashes in `data/manifests/YYYY-MM-DD.json`.

### Reconciliation (step 5)

5. For each newly extracted atom:

   **a. Deduplication check:**
   ```sql
   SELECT id FROM atoms
   WHERE kind = :kind AND body_hash = :body_hash AND retrieval_status = 'active';
   ```
   If match found with shared entity → skip atom, log in manifest as duplicate.

   **b. Correction candidate query:**
   - If kind is `observation`, skip correction detection entirely (observations coexist, never conflict).
   - Expand the new atom's entities through `entity_aliases` table.
   - Run the appropriate candidate query from `system/rules/reconciliation.md` based on kind:
     - Events/tasks: use `time_start`/`time_end` overlap query.
     - Resources: use `valid_from`/`valid_until` coverage overlap query (also match `resource_type`).
     - Timeless kinds (preference, decision, procedure, claim): entity + kind only.
   - Filter results: `body_hash != :new_body_hash` AND `observed_at < :new_observed_at`.
   - Apply kind-specific correction policy (see `system/rules/reconciliation.md` table).

   **c. If correction detected:**
   - Update old atom's Markdown: set `retrieval_status: corrected`, `corrected_by`, `corrected_at` in frontmatter; prepend correction notice to body.
   - Write new atom with `corrects: <old_id>`.
   - If correction chain depth reaches 4: flatten — update all prior versions' `corrected_by` to point directly to the new atom.

   **d. If ambiguous:** write both as `active`, log under "Needs review" in report.

   **e. Historical transitions:**
   - Query active atoms that meet kind-specific expiry criteria (see `system/rules/reconciliation.md` transition table).
   - Update their `retrieval_status` to `historical` and `updated_at` to now — in both Markdown frontmatter and SQLite.

   **f. Staleness flagging:**
   - Query: `SELECT * FROM atoms WHERE refresh_after < :today AND retrieval_status = 'active';`
   - Add results to `data/views/refresh-needed.md`. Do NOT change their `retrieval_status`.

### Index update (step 6)

6. Upsert all new and modified atoms into `data/indexes/knowledge.sqlite`:
   - Insert/update `atoms` table row.
   - Replace `atom_entities` rows for the atom.
   - Replace `atom_tags` rows for the atom.
   - If new entity aliases were discovered during extraction, insert into `entity_aliases`.

### View generation (step 7)

7. Generate/update views by querying SQLite:

   **`data/views/agenda-next-14-days.md`:**
   ```sql
   SELECT * FROM atoms
   WHERE retrieval_status = 'active'
     AND kind IN ('event', 'task')
     AND (time_start BETWEEN :today AND :today_plus_14
          OR time_due BETWEEN :today AND :today_plus_14)
   ORDER BY COALESCE(time_start, time_due);
   ```

   **`data/views/open-loops.md`:**
   ```sql
   SELECT * FROM atoms
   WHERE retrieval_status = 'active'
     AND kind = 'task'
     AND status IN ('open', 'blocked');
   ```

   **`data/views/recent-changes.md`:**
   ```sql
   SELECT * FROM atoms
   WHERE updated_at >= :today_minus_7
   ORDER BY updated_at DESC;
   ```

   **`data/views/refresh-needed.md`:**
   ```sql
   SELECT * FROM atoms
   WHERE retrieval_status = 'active'
     AND refresh_after < :today;
   ```

### Reporting (steps 8–9)

8. Write report to `data/reports/YYYY-MM-DD.md` using the output format below.
9. If retrieval indexing is enabled, follow `system/runbooks/reindex.md`; otherwise report that reindex is pending.

## Extraction policy

Extract atom kinds based on `system/taxonomy/atom-kinds/`, not topics. Each atom kind file defines:
- Required fields for that kind.
- What qualifies as that kind.
- Examples.

When uncertain about classification, prefer `observation` (the staging kind) over forcing into a more specific kind.

## Output report format

```text
# Daily Knowledge Report — YYYY-MM-DD

## Summary
- Sources scanned: <count>
- Source records written: <count>
- Atoms written: <count>
- Atoms corrected: <count>
- Atoms transitioned to historical: <count>
- Atoms deduplicated (skipped): <count>
- Entity aliases added: <count>
- Views updated: <list>

## Corrections applied
- <old_atom_id> → <new_atom_id> (reason)

## Needs review (ambiguous matches)
- <atom_id>: <description of ambiguity>

## Needs follow-up
- <description>

## Errors/skipped sources
- <source_spec>: <reason skipped>
```
