# workspace/AGENTS.md — Mutable Local Configuration

This folder contains local configuration for one installation of the personal memory index.

## Guardrails

- **Role:** mutable local configuration guide.
- **Mutation:** editable only through runbooks, primarily `system/runbooks/add-source.md`.
- **Allowed writes:** `workspace/source-specs/` for local source specs.
- **Do not:** store generated source records, atoms, reports, or indexes in `workspace/`.
- **Before acting:** validate whether the task is a local configuration change or a framework change.
- **Failure mode:** if a requested source needs new framework capabilities, write the local spec as `status: draft` and report the missing capability.

Agents may update this folder only by following runbooks under `system/runbooks/`.

## Source specs

Local source ingestion specs live under:

```text
workspace/source-specs/
```

To add a new source, copy `system/examples/source-spec-template.md` into `workspace/source-specs/<name>.md` and fill it in.

To disable a source, edit its spec to `status: disabled` or delete the local spec if the operator asks.

## Daily librarian behavior

1. Read `system/GUARDRAILS.md` and `system/RULES.md`.
2. Read all local `workspace/source-specs/*.md` files.
3. For each spec with `status: enabled`, follow its collection contract.
4. Write normalized source records under `data/sources/`.
5. Extract atoms under `data/atoms/`.
6. Record results under `data/manifests/` and `data/reports/`.

## Git policy

Most files under `workspace/` are local and ignored by git. The tracked files here only document how local configuration should work.
