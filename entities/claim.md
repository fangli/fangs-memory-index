# claim

A factual or opinion-like statement that may be useful later.

## Required fields

- `kind: claim`
- `statement`
- `source_refs`
- `confidence`

## Notes

Claims can expire, conflict, or be revised. Store `valid_from`, `valid_until`, and `refresh_after` when applicable.

## Examples

- A current policy page says a specific option exists.
- The operator prefers a specific architecture approach.
- An organization uses a specific program or process.
