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

CREATE INDEX IF NOT EXISTS idx_atoms_retrieval ON atoms(retrieval_status);
CREATE INDEX IF NOT EXISTS idx_atoms_kind ON atoms(kind);
CREATE INDEX IF NOT EXISTS idx_atoms_time_start ON atoms(time_start) WHERE time_start IS NOT NULL;
CREATE INDEX IF NOT EXISTS idx_atoms_time_end ON atoms(time_end) WHERE time_end IS NOT NULL;
CREATE INDEX IF NOT EXISTS idx_atoms_time_due ON atoms(time_due) WHERE time_due IS NOT NULL;
CREATE INDEX IF NOT EXISTS idx_atoms_valid_until ON atoms(valid_until) WHERE valid_until IS NOT NULL;
CREATE INDEX IF NOT EXISTS idx_atoms_refresh_after ON atoms(refresh_after) WHERE refresh_after IS NOT NULL;
CREATE INDEX IF NOT EXISTS idx_atoms_corrected_by ON atoms(corrected_by);
CREATE INDEX IF NOT EXISTS idx_entities_entity ON atom_entities(entity);
```

## `retrieval_status` values

| Value | Meaning |
|-------|---------|
| `active` | Current truth, forward-looking or timeless |
| `corrected` | Superseded by a newer version — still in indexed files but marked with inline correction notice |
| `historical` | Past fact, true for its time period, not subject to correction unless factually wrong |

## Rebuild

The index can be fully rebuilt by parsing all `data/atoms/**/*.md` frontmatter. The Markdown files are the source of truth; SQLite is derived.

## Not for retrieval

The retrieval agent does not query SQLite. It uses semantic/keyword search over Markdown files. SQLite is an internal tool for the indexing agent only.
