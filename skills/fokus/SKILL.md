---
name: fokus
description: Manage the user's tasks, calendar events, notes, objectives (goals), projects, reminders, and AI auto-scheduling in Fokus via the fokus-mcp CLI. Use when the user mentions their tasks, todos, schedule, agenda, calendar, day planning, notes, goals, projects, or Fokus.
license: MIT
compatibility: Requires Node.js 20+ and the fokus-mcp CLI on PATH; network access to a Fokus API
metadata:
  {
    "openclaw":
      {
        "emoji": "🎯",
        "homepage": "https://getfokus.com",
        "requires": { "bins": ["fokus-mcp"] },
        "install":
          [
            {
              "id": "npm",
              "kind": "node",
              "package": "@fokus-app/mcp",
              "bins": ["fokus-mcp"],
              "label": "Install the fokus-mcp CLI (npm)"
            }
          ]
      }
  }
---

# Fokus — AI productivity backend

Fokus (https://getfokus.com) is the user's productivity system: tasks, calendar, notes,
objectives (goals), projects ("buckets"), reminders, and an AI auto-scheduler. You operate it
through the `fokus-mcp` CLI, which handles authentication, workspaces, and converts note
content between markdown and Fokus's internal format.

## Setup check (run once per session)

```bash
fokus-mcp whoami
```

- Logged in → you'll see the user, API URL, and active workspace. Proceed.
- Not logged in → tell the user to run `fokus-mcp login` **themselves in a terminal** (it
  prompts interactively for email/password, or `fokus-mcp login --oauth google` for browser
  sign-in). Never ask for their password in chat.
- Credentials expire after ~7 days without use; the fix is the same `fokus-mcp login`.
- Self-hosted Fokus: set `FOKUS_API_URL` (e.g. `http://localhost:3000`) or use
  `fokus-mcp login --api-url <url>`.

## How to call tools

```bash
fokus-mcp tools                          # list all tools (name + summary)
fokus-mcp tools --json                   # machine-readable, includes JSON-schema inputs
fokus-mcp tool <name> '<json-args>'      # invoke a tool
fokus-mcp tool <name> -                  # read JSON args from stdin (for long markdown content)
```

Output is the tool's text result on stdout. Failures print a friendly message to stderr and
exit non-zero. Invalid args print the expected parameters — read them and retry.

Examples:

```bash
fokus-mcp tool get_current_datetime
fokus-mcp tool list_tasks '{"status":"open","limit":10}'
fokus-mcp tool create_task '{"title":"Finish Q3 report","priority":"high","estimatedTime":120,"dueDate":"2026-07-17T17:00:00+02:00"}'
printf '%s' '{"title":"Meeting notes","content":"# Design sync\n\n- new onboarding flow\n- ship next week"}' | fokus-mcp tool create_note -
```

## Core rules

1. **Dates:** always call `get_current_datetime` before interpreting "today", "tomorrow",
   "Friday". Send ISO 8601 with a UTC offset (`2026-07-15T14:00:00+02:00`).
2. **List before create** — avoid duplicates; reuse existing buckets/tags/objectives.
3. **Destructive tools** (`delete_*`, `update_note` with `content`) are permanent. Confirm
   with the user first unless they explicitly asked for the deletion.
4. **Enums:** task `priority` is `low|medium|high`; task `energyLevel` is
   `light|moderate|extensive`; objective `period` is `weekly|monthly|quarterly|yearly`.
5. **Markdown in, markdown out:** `description` (tasks/events), note `content`, and objective
   descriptions accept markdown; reads return markdown.
6. **Workspaces:** all tools operate in the active workspace. Check with `list_workspaces`;
   switch persistently with `fokus-mcp workspace <id|name>` (the `set_active_workspace` tool
   only lasts for a single CLI invocation, so don't use it from scripts).

## Tool cheat-sheet

Full parameter reference: [references/tools.md](references/tools.md).
Data-model conventions: [references/conventions.md](references/conventions.md).

- **Meta:** `get_current_user`, `get_current_datetime`, `list_workspaces`
- **Tasks:** `list_tasks`, `get_task`, `create_task`, `bulk_create_tasks` (≤25),
  `update_task`, `complete_task`, `delete_task`
- **Calendar:** `list_events`, `get_event`, `create_event`, `update_event`, `delete_event`,
  `list_calendars`
- **Notes:** `list_notes`, `get_note`, `create_note`, `update_note` (`appendContent` to add,
  `content` to replace), `delete_note`
- **Projects:** `list_buckets`, `create_bucket`, `update_bucket`, `delete_bucket`
- **Goals:** `list_objectives`, `get_objective`, `create_objective`, `update_objective`,
  `rollover_objective`, `delete_objective`
- **Tags & reminders:** `list_tags`, `create_tag`, `list_reminders`, `create_reminder`,
  `update_reminder`, `delete_reminder`
- **Overview:** `search` (cross-type by title/content), `get_schedule` (agenda for a range)
- **Guide:** `search_fokus_guide` (answers how-to questions about Fokus itself from the user guide)
- **Auto-scheduling:** `auto_schedule_tasks` → poll `get_scheduling_job` →
  `apply_scheduling_job` (or `cancel_scheduling_job`)

## Common workflows

**"What's on my plate today / this week?"**

```bash
fokus-mcp tool get_current_datetime
fokus-mcp tool get_schedule '{"startDate":"2026-07-10","endDate":"2026-07-10"}'
```

**Capture a task from conversation** — infer sensible `priority`, `energyLevel`, and
`estimatedTime` (minutes) from context; link `bucketId`/`objectiveId` when the user mentions a
project or goal:

```bash
fokus-mcp tool create_task '{"title":"Prepare investor deck","priority":"high","energyLevel":"extensive","estimatedTime":90,"dueDate":"2026-07-14T17:00:00+02:00"}'
```

**Auto-schedule the week** (asynchronous job — poll until `completed`, then apply):

```bash
fokus-mcp tool auto_schedule_tasks '{"rangeStart":"2026-07-13T00:00:00+02:00","rangeEnd":"2026-07-18T00:00:00+02:00"}'
fokus-mcp tool get_scheduling_job '{"jobId":"<id from previous call>"}'   # repeat every few seconds
fokus-mcp tool apply_scheduling_job '{"jobId":"<id>"}'
```

Summarize the proposed assignments to the user before applying.

**Weekly review:** `list_objectives` (scope `current`) → for unfinished ones ask whether to
`rollover_objective` → `list_tasks` with `status:"open"` and past `dueBefore` to surface
overdue work.

## Troubleshooting

| Symptom | Fix |
| --- | --- |
| `Not logged in` / auth errors | User runs `fokus-mcp login` in a terminal |
| `Permission denied ... different workspace` | `fokus-mcp workspace` to list, then switch |
| `Unknown tool` | `fokus-mcp tools` for the exact names |
| Invalid args (exit 1) | The error lists the expected parameters — fix and retry |
| Rate limited (HTTP 429) | Wait a moment and retry |
