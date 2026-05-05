# system/taxonomy/AGENTS.md — Atom Kind Registry

This folder defines information categories/kinds. These are not topic folders.

Daily ingestion and future tools should read all enabled Markdown files in `system/taxonomy/atom-kinds/` to understand how to classify extracted information.

## Add or change a category

Changing taxonomy is framework work. Do it only when the operator explicitly asks to change the framework.

1. Copy `system/templates/atom-record.md` if helpful.
2. Create or edit `system/taxonomy/atom-kinds/<kind>.md`.
3. Define purpose, required fields, temporal behavior, examples, and extraction rules.
4. Update `system/SCHEMAS.md` only if the shared schema changes.

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
