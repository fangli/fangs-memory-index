---
source_id: runtime.agent.session_logs
status: example
kind: session_logs
owner: operator
collection_mode: passive
frequency: daily
lookback: previous_local_day
trust: mixed
allowed_actions:
  - read_local_files
writes_allowed:
  - data/sources
  - data/atoms
dedupe_key:
  - source_id
  - origin.id
  - hash
freshness_policy:
  default: unknown
outputs:
  - source_record
  - atoms
  - view_updates
---

# Agent Session Logs

## Guardrails

- **Role:** example source spec.
- **Mutation:** read-only example; copy to `workspace/source-specs/` before local edits.
- **Allowed writes:** examples authorize no writes until copied into a local spec.
- **Do not:** treat `status: example` as enabled ingestion.
- **Before acting:** verify a local copied spec exists with `status: enabled`.
- **Failure mode:** if only the example exists, skip ingestion and report that no local source spec is enabled.

## Purpose

Extract reusable information from an agent runtime's conversations for the previous local calendar day.

## Collection contract

- Read session/transcript logs from the local runtime's configured transcript directory.
- Use the operator's configured local day boundaries.
- Include user messages, assistant answers, tool calls that reveal source URLs/files, and final summaries.
- Ignore pure tests, silent/no-op responses, routine no-op scheduled jobs, raw dream/reflection prose unless it contains concrete facts, and obvious duplicate retries.

## Extraction hints

- User corrections often indicate high-value preferences or decisions.
- Assistant research may contain reusable resources broader than the direct answer.
- File/document summaries can yield events, tasks, resources, and claims.
- Scheduled job outputs can yield operational observations, but should not pollute personal/work agenda unless concrete.

## Output requirements

- One source record per meaningful session or per coherent topic within a large session.
- Atoms should cite the session source record and line/session id where possible.
