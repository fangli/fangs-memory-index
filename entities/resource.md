# resource

A reusable source, page, file, dataset, schedule, policy, menu, or tool output.

## Required fields

- `kind: resource`
- `title`
- `resource_type`
- `coverage` or `observed_at`
- `source_refs`

## Retrieval behavior

Use resources to avoid repeated research. Refresh if outside coverage or after `refresh_after`.

## Examples

- Monthly lunch schedule.
- Airline upgrade policy snapshot.
- School mascot PDF.
