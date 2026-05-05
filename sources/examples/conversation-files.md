---
source_id: runtime.conversation_files
status: example
kind: files
owner: operator
collection_mode: passive
frequency: daily
lookback: previous_local_day
trust: primary
outputs:
  - source_record
  - atoms
---

# Conversation Files and Attachments

## Purpose

Capture reusable information from documents, images, audio transcripts, and other files shared in conversations.

## Collection contract

- Detect file references in session logs.
- Prefer extracted text already present in the session when available.
- If the file content is not available in the transcript, record a source placeholder with path/id and mark extraction incomplete.

## Extraction hints

- Flyers often contain events, tasks, deadlines, and resource links.
- Documents summarized in chat should be captured as source records with the summary and any source text available.
