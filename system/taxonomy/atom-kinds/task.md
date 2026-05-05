# task

A pending, completed, or implied action someone may need to do.

## Required fields

- `kind: task`
- `title`
- `status: open | done | blocked | maybe | unknown`
- `source_refs`

## Optional fields

- `due`
- `owner`
- `entities`
- `priority`
- `blocks`

## Retrieval behavior

Agenda/open-loop queries should include open tasks due in or near the requested range. Include undated tasks in open-loop views, not necessarily agenda views.
