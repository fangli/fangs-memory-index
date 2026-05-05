# resource

A reusable source, page, file, dataset, schedule, policy, menu, or tool output.

## Required fields

- `kind: resource`
- `title`
- `resource_type`
- `source_refs`

## Coverage

Resources use `freshness.valid_from` and `freshness.valid_until` to express their coverage period. These map to the `valid_from` and `valid_until` columns in SQLite.

For resources without a clear coverage period, `observed_at` alone is sufficient (it's required on all atoms).

The reconciliation rule for resources checks overlapping coverage via `valid_from`/`valid_until`, not `time.start`/`time.end` (which are for events/tasks).

## Retrieval behavior

Use resources to avoid repeated research. Refresh if outside coverage or after `refresh_after`.

## Examples

- Monthly lunch schedule.
- Airline upgrade policy snapshot.
- School mascot PDF.
