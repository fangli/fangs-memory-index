# event

An occurrence with time relevance.

Use for meetings, school events, deadlines with date ranges, special days, appointments, trips, activities, and agenda items.

## Required fields

- `kind: event`
- `title`
- one of `time.start`, `time.end`, `time.date`, or `time.expression`
- `source_refs`

## Retrieval behavior

Agenda queries should include events whose time overlaps the requested fuzzy range.

## Examples

- Teacher Appreciation Week, 2026-05-04 to 2026-05-08.
- Family Day Picnic, 2026-05-29 15:30 to 17:00.
- Meeting from email invitation.
