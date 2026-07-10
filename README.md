# Fokus agent skills

Connect AI agent frameworks to [Fokus](https://getfokus.com) — tasks, calendar, notes,
objectives, and AI auto-scheduling, driven by your personal AI assistant.

Everything here builds on the **`fokus-mcp` CLI** (`@fokus-app/mcp`, from the `mcp-server`
repo). It owns authentication (login once, tokens auto-refresh), workspace selection, and
markdown conversion, and offers the same 44 tools two ways:

- `fokus-mcp` (no subcommand) — an MCP server over stdio
- `fokus-mcp tool <name> '<json>'` — direct CLI invocation, used by the skill

## What's here

```
skills/fokus/          The "fokus" skill (agentskills.io format, OpenClaw-compatible)
  SKILL.md             Agent instructions
  references/          Full tool + data-model reference (loaded on demand)
docs/openclaw.md       Setup guide for OpenClaw
docs/hermes.md         Setup guide for Hermes Agent
```

## Which integration route?

| Framework | Recommended | Also works |
| --- | --- | --- |
| [OpenClaw](https://openclaw.ai) | **Skill** — `openclaw skills install .../skills/fokus` | MCP server |
| [Hermes Agent](https://github.com/nousresearch/hermes-agent) | **MCP server** — `mcp_servers:` entry in `~/.hermes/config.yaml` | Skill in `~/.hermes/skills/` |
| Claude Code / Claude Desktop / Cursor | MCP server (see the mcp-server repo README) | — |

Pick one route per framework — installing both duplicates the same capabilities.

## Quick start

```bash
npm install -g @fokus-app/mcp     # (until npm publish: build mcp-server and `npm link`)
fokus-mcp login
fokus-mcp workspace               # pick your default workspace
```

Then follow [docs/openclaw.md](docs/openclaw.md) or [docs/hermes.md](docs/hermes.md).

## Requirements & limits

- Node.js 20+ where the agent runs; network access to the Fokus API (cloud or self-hosted
  via `FOKUS_API_URL` / `fokus-mcp login --api-url`).
- Login is interactive and must be done by a human once; credentials expire after ~7 days
  without use. Unattended/headless API-key auth is future work.

## License

MIT
