# Personal Memory Index

A generic, agent-friendly knowledge indexing system for preserving reusable information from conversations, files, email-like messages, web research, calendar-like events, and other arbitrary sources.

The project is designed for any AI-agent runtime that can read files, write files, run scheduled jobs, and optionally expose a retrieval/search index. It is not tied to a specific agent platform.

## Core idea

Do not organize knowledge by static topic folders. Topics are unpredictable.

Organize by information behavior:

- **Source** — where information came from.
- **Atom** — a compact reusable unit extracted from sources.
- **View** — a generated rollup for common fuzzy question shapes.
- **Entity/category definition** — shared rules for atom kinds and routing behavior.
- **Source spec** — pluggable ingestion requirements.

## Path convention

All runbooks refer to the project root as:

```text
knowledge_base_dir
```

Every other path is relative to that base directory.

Example:

```text
knowledge_base_dir/library/sources
knowledge_base_dir/library/atoms
knowledge_base_dir/sources
knowledge_base_dir/entities
```

## Layout

```text
knowledge_base_dir/
  AGENTS.md             # operator instructions for agents
  SPEC.md               # architecture/specification
  RULES.md              # extraction, freshness, provenance, mutation rules
  INSTALLATION.md       # setup runbook; setup is not assumed complete
  RUNBOOKS.md           # operational runbook index
  entities/             # generic atom kind/category definitions
  sources/              # local pluggable source specs; personal specs are gitignored
  sources/examples/     # reusable example source specs
  schemas/              # source/atom/view schema docs
  runbooks/             # executable-by-agent procedures
  templates/            # copyable templates
  library/              # generated user knowledge; gitignored by default
  tools/                # future helper scripts/docs
```

## Generated data policy

The repository tracks the framework, schemas, runbooks, templates, and generic examples.
It intentionally does **not** track local user data generated under `library/` or local source specs added under `sources/`.

## Current status

This repository is documentation/runbook-first. Creating the files does not install a cron job, configure a retrieval index, or collect external data.
