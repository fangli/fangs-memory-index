# SQLite Index Schema

The deterministic index lives at `data/indexes/knowledge.sqlite`. It is a derived projection of atom Markdown files — rebuildable from `data/atoms/` at any time.

## Purpose

The index serves the **smart indexing agent** during daily ingestion:

- Fast lookup of existing atoms by entity, kind, and time range
- Correction candidate detection during reconciliation
- Efficient view generation (agenda, open loops, refresh-needed)
- Deduplication checks

## Schema

```sql
CREATE TABLE IF NOT EXISTS atoms (
    id              TEXT PRIMARY KEY,
    kind            TEXT NOT NULL,
    title           TEXT,
    status          TEXT,
    confidence      REAL,
    retrieval_status TEXT NOT NULL DEFAULT 'active',

    -- Time
    time_start      TEXT,
    time_end        TEXT,
    time_due        TEXT,
    time_expression TEXT,

    -- Freshness
    observed_at     TEXT NOT NULL,
    valid_from      TEXT,
    valid_until     TEXT,
    refresh_after   TEXT,

    -- Correction chain
    corrects        TEXT,
    corrected_by    TEXT,
    corrected_at    TEXT,

    -- Deduplication
    body_hash       TEXT,

    -- Kind-specific
    resource_type   TEXT,

    -- Provenance
    source_refs     TEXT NOT NULL,
    file_path       TEXT NOT NULL,

    -- Metadata
    created_at      TEXT NOT NULL,
    updated_at      TEXT NOT NULL
);

CREATE TABLE IF NOT EXISTS atom_entities (
    atom_id TEXT NOT NULL REFERENCES atoms(id),
    entity  TEXT NOT NULL,
    PRIMARY KEY (atom_id, entity)
);

CREATE TABLE IF NOT EXISTS atom_tags (
    atom_id TEXT NOT NULL REFERENCES atoms(id),
    tag     TEXT NOT NULL,
    PRIMARY KEY (atom_id, tag)
);

CREATE TABLE IF NOT EXISTS entity_aliases (
    canonical TEXT NOT NULL,
    alias     TEXT NOT NULL,
    PRIMARY KEY (canonical, alias)
);

CREATE INDEX IF NOT EXISTS idx_atoms_retrieval ON atoms(retrieval_status);
CREATE INDEX IF NOT EXISTS idx_atoms_kind ON atoms(kind);
CREATE INDEX IF NOT EXISTS idx_atoms_time_start ON atoms(time_start) WHERE time_start IS NOT NULL;
CREATE INDEX IF NOT EXISTS idx_atoms_time_end ON atoms(time_end) WHERE time_end IS NOT NULL;
CREATE INDEX IF NOT EXISTS idx_atoms_time_due ON atoms(time_due) WHERE time_due IS NOT NULL;
CREATE INDEX IF NOT EXISTS idx_atoms_valid_until ON atoms(valid_until) WHERE valid_until IS NOT NULL;
CREATE INDEX IF NOT EXISTS idx_atoms_refresh_after ON atoms(refresh_after) WHERE refresh_after IS NOT NULL;
CREATE INDEX IF NOT EXISTS idx_atoms_corrected_by ON atoms(corrected_by);
CREATE INDEX IF NOT EXISTS idx_atoms_body_hash ON atoms(body_hash) WHERE body_hash IS NOT NULL;
CREATE INDEX IF NOT EXISTS idx_entities_entity ON atom_entities(entity);
CREATE INDEX IF NOT EXISTS idx_entity_aliases_alias ON entity_aliases(alias);
```

## Entity Normalization

Entities stored in `atom_entities` are normalized before insertion:

1. Lowercase the string.
2. Strip leading/trailing whitespace.
3. Collapse internal whitespace to a single space.

The `entity_aliases` table maps known alternate forms to a canonical entity. During reconciliation, entity overlap checks expand each entity through its aliases before comparing.

Example aliases:

| canonical | alias |
|-----------|-------|
| fang | fang li |
| luna | luna li |
| shanghai | pvg |

The indexing agent maintains this table. When a new entity is extracted that appears to be an alternate form of an existing canonical entity (same person, place, or thing), add an alias row. When uncertain, do not alias — treat as separate entities and log for review.

## `body_hash`

SHA-256 hex digest of the atom body content (everything after the closing `---` of frontmatter), stripped of leading/trailing whitespace and with line endings normalized to `\n`. Used for fast deduplication: if a new atom has the same `body_hash` as an existing active atom of the same kind, it is a duplicate.

## `retrieval_status` values

| Value | Meaning |
|-------|---------|
| `active` | Current truth, forward-looking or timeless |
| `corrected` | Superseded by a newer version — still in indexed files but marked with inline correction notice |
| `historical` | Past fact, true for its time period, not subject to correction unless factually wrong |

## Rebuild

The index can be fully rebuilt by parsing all `data/atoms/**/*.md` frontmatter and body. The Markdown files are the source of truth; SQLite is derived.

Rebuild procedure:

1. Create the database using the schema above if it does not exist.
2. Parse each `data/atoms/**/*.md` file: extract frontmatter fields and compute `body_hash`.
3. Upsert into `atoms`, `atom_entities`, `atom_tags`.
4. Rebuild `entity_aliases` from any `data/indexes/entity-aliases.yaml` if present, or leave existing aliases intact.

## Not for retrieval

The retrieval agent does not query SQLite directly. It uses semantic/keyword search over Markdown files. SQLite is an internal tool for the indexing agent only.

However, the retrieval agent benefits indirectly: corrected atoms contain inline notices that point to the current version, and views (generated from SQLite queries) pre-filter active atoms.
