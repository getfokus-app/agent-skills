# Fokus data-model conventions

## Dates and timezones

- Every date parameter is ISO 8601. Include a UTC offset for times: `2026-07-15T14:00:00+02:00`.
- Call `get_current_datetime` first when the user says "today", "tomorrow", "Friday", "next
  week" — it returns the current moment in the **user's** timezone, which is rarely UTC.
- Date-only values (`startDate`, objective `start`/`end`) accept `YYYY-MM-DD`.

## Task fields worth understanding

- `dueDate` = the deadline. `doDate` = the day the user plans to work on it. `start`/`end` =
  a concrete calendar slot (setting `start` pins the task; the auto-scheduler won't move it).
- `deadlineMode`: `soft` (default) lets the auto-scheduler place work after the due date if
  needed; `hard` never does.
- `estimatedTime` is minutes; the auto-scheduler uses it to size calendar slots.
- `priority`: `low | medium | high`. `energyLevel`: `light | moderate | extensive` — how
  demanding the work is; the scheduler matches extensive work to high-energy hours.
- Subtasks: `parentId`. Dependencies: `blockedBy` (array of task ids, update_task only).
- Recurring tasks/events use an RRULE set **including DTSTART**, newline-separated:
  `"DTSTART:20260713T090000Z\nRRULE:FREQ=WEEKLY;BYDAY=MO,WE"`.

## Rich text is markdown at the CLI boundary

Fokus stores rich text as TipTap JSON internally. The `fokus-mcp` CLI converts both ways:
you always **send** markdown (`description`, note `content`, `appendContent`) and **receive**
markdown. Supported: headings, bold/italic, bullet/numbered/task lists, tables, code blocks,
blockquotes, links. Content larger than ~100k characters is truncated on read.

## Workspaces

- All data is workspace-scoped. The CLI sends the persisted active workspace automatically.
- `fokus-mcp workspace` lists workspaces (active marked with `*`); `fokus-mcp workspace
  <id|name>` switches the persisted default. Do this instead of the `set_active_workspace`
  tool, which only affects a single CLI process.
- A 403 "different workspace" error usually means the resource lives in another workspace.

## Pagination and sorting

- List tools return a header line (`Workspace: X — N items (page 1/3, showing 20)`) followed
  by a JSON array. Fetch further pages with `page`.
- `sort` takes a field name, `-` prefix for descending (e.g. `-createdAt`, `dueDate`).

## Objects and ids

- Ids are Mongo ObjectIds (24 hex chars) returned by every create/list tool as `id`.
- Buckets are Fokus's "projects": they group tasks, events, and notes, and can nest.
- Objectives are period-bound goals (`weekly|monthly|quarterly|yearly`); tasks link to them
  via `objectiveId`, and `get_objective` shows progress over linked tasks.
- Reminders attach to an activity (task or event) via `activityId`.

## Error behavior

`fokus-mcp tool` exits non-zero on failure and prints a human-readable reason to stderr:
authentication required, permission denied (workspace mismatch), not found (bad id), rate
limited (retry after a pause), or validation errors listing the expected parameters.
