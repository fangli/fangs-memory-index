---
source_id: runtime.agent.session_logs
status: example
kind: session_logs
owner: operator
collection_mode: passive
frequency: daily
lookback: previous_local_day
trust: mixed
outputs:
  - source_record
  - atoms
  - view_updates
---

# Agent Session Logs

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
