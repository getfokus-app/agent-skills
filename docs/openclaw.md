# Using Fokus with OpenClaw

Two ways to connect [OpenClaw](https://openclaw.ai) to Fokus. Both need the `fokus-mcp` CLI
logged in once; pick **one** (installing both duplicates the same capabilities in the
agent's context).

- **Skill (recommended):** the `fokus` skill teaches the agent Fokus's data model, workflows,
  and best practices, and calls the CLI through OpenClaw's normal shell tooling.
- **MCP server:** raw tool access via the Model Context Protocol, no skill instructions.

## Prerequisites

1. Node.js 20+ on the machine running OpenClaw.
2. Install the CLI:

   ```bash
   npm install -g @fokus-app/mcp
   ```

3. Log in (one-time; tokens auto-refresh while used, and expire after ~7 idle days):

   ```bash
   fokus-mcp login                       # email/password
   fokus-mcp login --oauth google        # or browser sign-in (also: microsoft, apple)
   fokus-mcp whoami                      # verify
   fokus-mcp workspace                   # optional: pick the default workspace
   ```

   Self-hosted Fokus: add `--api-url http://your-host:3000`.

## Option A — install the skill

From a checkout of this repo:

```bash
openclaw skills install /path/to/agent-skills/skills/fokus
```

Or directly from git:

```bash
openclaw skills install git:<owner>/agent-skills#skills/fokus
```

No `apiKey` or `env` entries are needed in `~/.openclaw/openclaw.json` — authentication lives
in the fokus-mcp credential store (`~/.config/fokus-mcp/credentials.json`), not in
environment variables. The skill gates on the `fokus-mcp` binary being on `PATH`
(`metadata.openclaw.requires.bins`); if it's missing, OpenClaw's installer hint offers the
npm install.

To scope the skill to specific agents, add it to the allowlist in `openclaw.json`:

```json5
{
  agents: {
    defaults: { skills: ["fokus"] }
  }
}
```

Then ask your agent things like:

- "Add a task to finish the Q3 report by Friday, high priority"
- "What does my week look like?"
- "Schedule my open tasks for this week" (auto-scheduler)
- "Take a note: design sync decided on the new onboarding flow"

## Option B — MCP server

Point OpenClaw's MCP support at the same CLI in server mode (running `fokus-mcp` with no
subcommand serves MCP over stdio):

```json5
// ~/.openclaw/openclaw.json (see OpenClaw's MCP docs for the current key layout)
{
  mcp: {
    servers: {
      fokus: { command: "fokus-mcp" }
    }
  }
}
```

This exposes the same 44 tools plus two MCP prompts (`daily_planning`, `weekly_review`), but
without the skill's usage guidance.

To use the **hosted** MCP server instead of the local binary (if your OpenClaw build supports
remote MCP), mint a token with `fokus-mcp token create` and point at the URL:

```json5
{
  mcp: {
    servers: {
      fokus: {
        url: "https://api.getfokus.com/mcp",
        headers: { Authorization: "Bearer <token>" }
      }
    }
  }
}
```

## Troubleshooting

| Symptom | Fix |
| --- | --- |
| Skill greyed out / not offered | `fokus-mcp` not on the PATH OpenClaw uses — install globally, restart the gateway |
| "Not logged in" from tools | Run `fokus-mcp login` in a terminal (interactive; can't be done by the agent) |
| Auth works, wrong data | Check the active workspace: `fokus-mcp workspace` |
| Self-hosted API ignored | Log in with `--api-url`; the credential store is keyed per API URL |
