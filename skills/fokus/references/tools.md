# Fokus tool reference

All 45 tools invokable via `fokus-mcp tool <name> '<json-args>'`. Required parameters are
marked `*`. For exact JSON schemas run `fokus-mcp tools --json`.

Shared conventions: dates are ISO 8601 with a UTC offset; `estimatedTime` and
`offsetMinutes` are minutes; `description`/`content` are markdown; list tools paginate with
`page`/`limit` and support `sort` (prefix `-` for descending). See
[conventions.md](conventions.md).

## Meta

### get_current_user
Logged-in user profile: name, email, timezone, settings. Learn the timezone before working with dates. No params.

### get_current_datetime
Current date/time in the user's timezone. Call before interpreting relative dates. No params.

### list_workspaces
The user's workspaces with their role in each; the active one is marked. No params.

### set_active_workspace
Switch workspace **for this invocation only** — pointless from the CLI, where every call is a
new process. Use `fokus-mcp workspace <id|name>` instead to change the persisted default.
- `workspace*` (string) — id, slug, or exact name

## Tasks

### list_tasks
Defaults to open tasks, newest first.
- `status` (`open|completed|all`, default `open`)
- `priority` (`low|medium|high`), `energyLevel` (`light|moderate|extensive`)
- `bucketId`, `parentId`, `calendarId`, `tagId` (strings)
- `titleContains` (string)
- `dueAfter`, `dueBefore`, `doDateFrom`, `doDateTo` (ISO dates)
- `recurring` (boolean: true = only recurring, false = exclude recurring)
- `sort` (fields: createdAt, title, doDate, dueDate, priority, start; default `-createdAt`), `page`, `limit`

### get_task
Full task details, description as markdown by default.
- `taskId*`, `includeDescription` (boolean, default true)

### create_task
Setting `start` pins the task to that slot; otherwise the auto-scheduler may place it.
- `title*`
- `description` (markdown), `priority` (`low|medium|high`), `energyLevel` (`light|moderate|extensive`)
- `estimatedTime` (minutes), `dueDate` (deadline), `doDate` (day the user plans to work on it)
- `start`, `end` (pin to a time slot), `deadlineMode` (`soft|hard`, default soft)
- `bucketId`, `objectiveId`, `parentId` (subtask of), `calendarId`, `tagIds` (array)
- `recurringPattern` (RRULE set incl. DTSTART, e.g. `"DTSTART:20260713T090000Z\nRRULE:FREQ=WEEKLY;BYDAY=MO"`)
- `excludeFromScheduling` (boolean)

### bulk_create_tasks
Up to 25 tasks in one call (e.g. breaking down a project); reports success/failure per task.
- `tasks*` (array of create_task-shaped objects)

### update_task
Only provided fields change; `description` replaces the whole description. Set `start`/`end`/`dueDate`/`doDate` to `null` to clear.
- `taskId*` + any create_task field, plus `isPinned` (boolean), `blockedBy` (array of task ids)

### complete_task
- `taskId*`, `completed` (boolean, default true; false un-completes)

### delete_task (destructive)
Permanent, removes subtask links too.
- `taskId*`

## Calendar events

### list_events
Sorted by start ascending.
- `startFrom`, `startTo` (ISO — the range you care about), `calendarId`, `bucketId`, `titleContains`, `sort`, `page`, `limit`

### get_event
- `eventId*`, `includeDescription` (boolean)

### create_event
Defaults to the user's primary calendar.
- `title*`, `start*`, `end*` (ISO with offset)
- `isAllDay` (boolean), `eventType` (`default|outOfOffice|focusTime|other`), `status` (`confirmed|tentative|cancelled`)
- `calendarId`, `bucketId`, `description` (markdown), `recurringPattern` (RRULE), `excludeFromScheduling`

### update_event
- `eventId*` + any create_event field except `recurringPattern`

### delete_event (destructive)
Also removes externally-synced events (Google/Outlook) at the source.
- `eventId*`

### list_calendars
Fokus-native and synced calendars; use the ids for events and filters. No params.

## Notes

### list_notes
Returns a markdown snippet per note. Auto-created task/event description notes are hidden unless filtering by `activityId`.
- `titleContains`, `contentContains`, `activityId` (task/event id), `tagId`, `includeDescriptions` (boolean), `updatedAfter` (ISO), `sort`, `page`, `limit`

### get_note
Full content as markdown.
- `noteId*`

### create_note
Markdown content: headings, lists, task lists, tables, code blocks, links all render in Fokus.
- `title*`, `content*` (markdown)
- `icon` (emoji), `bucketId`, `tagIds`, `activityId` (attach to a task/event), `objectiveId`

### update_note (destructive when replacing)
`content` REPLACES the whole body; prefer `appendContent` to add to the end.
- `noteId*`, `title`, `icon`, `content`, `appendContent`

### delete_note (destructive)
- `noteId*`

## Buckets (projects)

### list_buckets
Buckets group tasks/events/notes and can nest via parent. — `titleContains`, `page`, `limit`

### create_bucket
- `title*`, `color`, `parentId`, `defaultCalendarId`

### update_bucket
- `bucketId*`, `title`, `color`, `parentId`, `defaultCalendarId`

### delete_bucket (destructive)
Contained items are kept but lose the bucket assignment.
- `bucketId*`

## Objectives (goals)

### list_objectives
- `scope` (`current|history`, default current), `period` (`weekly|monthly|quarterly|yearly`), `status` (`open|completed|all`, default all), `titleContains`, `page`, `limit`

### get_objective
Includes description (markdown) and linked tasks.
- `objectiveId*`

### create_objective
`start`/`end` should span the period (e.g. Monday–Sunday for weekly).
- `title*`, `period*` (`weekly|monthly|quarterly|yearly`), `start*`, `end*`, `bucketId`, `description` (markdown)

### update_objective
- `objectiveId*`, `title`, `period`, `start`, `end`, `isCompleted` (boolean), `bucketId`

### rollover_objective
Carry an unfinished objective (and its open tasks) to the next period.
- `objectiveId*`, `targetPeriod` (object, defaults to the next period)

### delete_objective (destructive)
Linked tasks kept but unlinked.
- `objectiveId*`

## Tags & reminders

### list_tags
- `nameContains`, `limit`

### create_tag
- `name*`, `color`

### list_reminders
Default: upcoming only, soonest first.
- `activityId`, `upcomingOnly` (boolean), `limit`

### create_reminder
Absolute fire time; optionally record what it's relative to so it follows recurring activities.
- `activityId*` (task/event id), `date*` (ISO)
- `relativeTo` (`start|end|doDate|dueDate|scheduledStart`), `offsetMinutes` (integer, 0 or more: minutes before the `relativeTo` point, e.g. `15` = 15 min before)

### update_reminder
- `reminderId*`, `date`, `relativeTo`, `offsetMinutes`

### delete_reminder (destructive)
- `reminderId*`

## Search & schedule

### search
Cross-type search by title (and note content), grouped by type. Use list_* tools for filtered queries.
- `query*`, `types` (array, default all), `limit`

### get_schedule
Agenda for a range: events plus tasks planned (doDate) or scheduled (start), grouped by day. Ideal for "what does my day/week look like".
- `startDate*`, `endDate*` (ISO dates), `includeCompleted` (boolean)

## Fokus guide

### search_fokus_guide
Searches the official user guide (guide.getfokus.com) for how Fokus works: features, settings, integrations, add-ons, limits. Returns guide sections with a citable URL. Knows nothing about the user's own tasks, events or notes; use `search` or the list_* tools for those.
- `query*` (string, 3–300 characters), `limit` (integer 1–8, default 5)

## Auto-scheduling

Asynchronous job flow: start → poll → apply (or cancel).

### auto_schedule_tasks
Places unscheduled tasks into free slots in the range, respecting energy levels, priorities, deadlines, and working hours. Returns a job id.
- `rangeStart*`, `rangeEnd*` (ISO), `timeZone` (IANA, defaults to user timezone)

### get_scheduling_job
Status plus, when finished, the proposed assignments.
- `jobId*`

### apply_scheduling_job
Writes the proposal's start/end times to the calendar.
- `jobId*`

### cancel_scheduling_job
- `jobId*`
