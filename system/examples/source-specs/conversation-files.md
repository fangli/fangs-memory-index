---
source_id: runtime.conversation_files
status: example
kind: files
owner: operator
collection_mode: passive
frequency: daily
lookback: previous_local_day
trust: primary
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
---

# Conversation Files and Attachments

## Guardrails

- **Role:** example source spec.
- **Mutation:** read-only example; copy to `workspace/source-specs/` before local edits.
- **Allowed writes:** examples authorize no writes until copied into a local spec.
- **Do not:** treat `status: example` as enabled ingestion.
- **Before acting:** verify a local copied spec exists with `status: enabled`.
- **Failure mode:** if only the example exists, skip ingestion and report that no local source spec is enabled.

## Purpose

Capture reusable information from documents, images, audio transcripts, and other files shared in conversations.

## Collection contract

- Detect file references in session logs.
- Prefer extracted text already present in the session when available.
- If the file content is not available in the transcript, record a source placeholder with path/id and mark extraction incomplete.

## Extraction hints

- Flyers often contain events, tasks, deadlines, and resource links.
- Documents summarized in chat should be captured as source records with the summary and any source text available.
