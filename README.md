# Personal Memory Index

A generic, agent-friendly knowledge indexing system for preserving reusable information from conversations, files, email-like messages, web research, calendar-like events, and other arbitrary sources.

The project is designed for any AI-agent runtime that can read files, write files, run scheduled jobs, and optionally expose a retrieval/search index. It is not tied to a specific agent platform.

## Core idea

Do not organize knowledge by static topic folders. Topics are unpredictable.

Organize by information behavior:

- **Source** — where information came from.
- **Atom** — a compact reusable unit extracted from sources.
- **View** — a generated rollup for common fuzzy question shapes.
- **Taxonomy** — shared rules for atom kinds and routing behavior.
- **Source spec** — pluggable ingestion requirements.

## Path convention

All runbooks refer to the project root as:

```text
knowledge_base_dir
```

Every other path is relative to that base directory.

## Layout

```text
knowledge_base_dir/
  README.md
  AGENTS.md             # repository boundary guide
  LICENSE
  CHANGELOG.md

  system/               # static framework; read-only unless changing framework
    SPEC.md
    RULES.md
    INSTALLATION.md
    RUNBOOKS.md
    SCHEMAS.md
    runbooks/
    schemas/
    taxonomy/
    templates/
    examples/
    tools/

  workspace/            # mutable local configuration; mostly gitignored
    AGENTS.md
    source-specs/
      README.md
      *.md              # local ingestion specs, ignored by default

  data/                 # generated user knowledge; gitignored by default
    sources/
    atoms/
    views/
    manifests/
    indexes/
    reports/
```

## Mutation boundary

```text
system/     static framework files; agents read, do not modify by default
workspace/  mutable local configuration; agents may update via runbook
data/       generated user data; daily ingestion may update/index
```

## Generated data policy

The repository tracks the framework, schemas, runbooks, templates, and generic examples.
It intentionally does **not** track generated user data under `data/` or local source specs under `workspace/source-specs/`.

## Current status

This repository is documentation/runbook-first. Creating the files does not install a scheduled job, configure a retrieval index, or collect external data.
