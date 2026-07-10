# Using Fokus with Hermes Agent

Two ways to connect [Hermes Agent](https://github.com/nousresearch/hermes-agent) (Nous
Research) to Fokus. Both need the `fokus-mcp` CLI logged in once; pick **one**.

- **MCP server (recommended for Hermes):** native first-class integration — Hermes loads the
  44 Fokus tools with their schemas directly.
- **Skill:** the same `fokus` skill OpenClaw uses; Hermes supports the
  [agentskills.io](https://agentskills.io) format.

## Prerequisites

1. Node.js 20+ on the machine running Hermes.
2. Install the CLI:

   ```bash
   npm install -g @fokus-app/mcp
   ```

   > Until the package is published to npm, install from source instead:
   > `git clone <mcp-server repo> && cd mcp-server && npm ci && npm run build && npm link`

3. Log in (one-time; tokens auto-refresh while used, and expire after ~7 idle days):

   ```bash
   fokus-mcp login                       # email/password
   fokus-mcp login --oauth google        # or browser sign-in (also: microsoft, apple)
   fokus-mcp whoami                      # verify
   fokus-mcp workspace                   # optional: pick the default workspace
   ```

   Self-hosted Fokus: add `--api-url http://your-host:3000`.

## Option A — MCP server

Add to `~/.hermes/config.yaml`:

```yaml
mcp_servers:
  fokus:
    command: "fokus-mcp"
```

(Running `fokus-mcp` with no subcommand serves MCP over stdio.) For a self-hosted backend:

```yaml
mcp_servers:
  fokus:
    command: "fokus-mcp"
    env:
      FOKUS_API_URL: "http://your-host:3000"
```

Restart Hermes; the Fokus tools appear alongside the built-in toolsets, plus two MCP prompts
(`daily_planning`, `weekly_review`).

## Option B — skill

Copy (or symlink) the skill into Hermes's skills directory:

```bash
cp -r /path/to/agent-skills/skills/fokus ~/.hermes/skills/fokus
```

Browse with `/skills` or invoke with `/fokus`. The skill drives the same `fokus-mcp` CLI via
Hermes's terminal tool, so it works in any terminal backend that has Node and the CLI
installed (for Docker/remote backends, install and log in the CLI **inside** that
environment).

If you previously installed the skill in OpenClaw, `hermes claw migrate` imports it
automatically into `~/.hermes/skills/openclaw-imports/`.

## Example prompts

- "Add a task to finish the Q3 report by Friday, high priority"
- "What's on my calendar tomorrow?"
- "Create a weekly objective 'Ship v2 beta' and break it into tasks"
- "Auto-schedule my week"

## Troubleshooting

| Symptom | Fix |
| --- | --- |
| Tools missing after config edit | Restart Hermes; check `hermes mcp` shows the server connected |
| "Not logged in" from tools | Run `fokus-mcp login` in a terminal (interactive; can't be done by the agent) |
| Skill runs but `fokus-mcp: command not found` | The terminal backend (e.g. Docker) doesn't have the CLI — install it there, or use the MCP route |
| Auth works, wrong data | Check the active workspace: `fokus-mcp workspace` |
