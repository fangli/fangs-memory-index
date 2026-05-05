# event

An occurrence with time relevance.

Use for meetings, school events, deadlines with date ranges, special days, appointments, trips, activities, and agenda items.

## Required fields

- `kind: event`
- `title`
- one of `time.start`, `time.end`, or `time.expression`
- `source_refs`

## Time field mapping

When an event has a single date (e.g., "May 5"), store it as `time.start`. When it spans a range (e.g., "May 4 to May 8"), store both `time.start` and `time.end`.

The SQLite index maps these to `time_start` and `time_end` columns. The `time.expression` field is for fuzzy/recurring time descriptions that can't be resolved to exact dates (e.g., "every Tuesday").

## Retrieval behavior

Agenda queries should include events whose time overlaps the requested fuzzy range.

## Examples

- Teacher Appreciation Week, 2026-05-04 to 2026-05-08.
- Family Day Picnic, 2026-05-29 15:30 to 17:00.
- Meeting from email invitation.
