# entities/AGENTS.md — Atom Category Registry

This folder defines information categories/kinds. These are not topic folders.

Daily ingestion and future tools should read all enabled Markdown files in this folder to understand how to classify extracted information.

## Add a category

1. Copy `../templates/atom-record.md` if helpful.
2. Create `entities/<kind>.md`.
3. Define purpose, required fields, temporal behavior, examples, and extraction rules.
4. Update `../SCHEMAS.md` only if the shared schema changes.

## Existing canonical kinds

- `event`
- `task`
- `claim`
- `preference`
- `decision`
- `resource`
- `procedure`
- `observation`
- `view`
