# Fokus agent skills

Connect AI agent frameworks to [Fokus](https://getfokus.com) — tasks, calendar, notes,
objectives, and AI auto-scheduling, driven by your personal AI assistant.

There are two ways to connect, exposing the same 45 tools:

- **Hosted (remote) MCP** — a Streamable HTTP endpoint at `https://api.getfokus.com/mcp`.
  Nothing to install; the client authenticates with a bearer token. Best for Claude Code and
  any client that speaks remote MCP.
- **Local `fokus-mcp` CLI** (`@fokus-app/mcp`, from the `mcp-server` repo) — owns
  authentication (login once, tokens auto-refresh), workspace selection, and markdown
  conversion, and offers the tools two ways: `fokus-mcp` (no subcommand) is an MCP server
  over stdio, and `fokus-mcp tool <name> '<json>'` is direct CLI invocation (used by the skill).
  Best for frameworks that run a local binary, offline/self-hosted setups, and the skill route.

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
| Claude Code / Claude Desktop / Cursor | **Hosted MCP** — `claude mcp add --transport http` | Local stdio MCP server |

Pick one route per framework — installing both duplicates the same capabilities.

## Quick start — hosted (no install)

Mint a token (needs the CLI logged in once, or your app account), then point any remote-MCP
client at the hosted endpoint:

```bash
npm install -g @fokus-app/mcp
fokus-mcp login
fokus-mcp token create --label "Claude Code"   # prints a ready-to-paste command:
# claude mcp add --transport http fokus https://api.getfokus.com/mcp \
#   --header "Authorization: Bearer <token>"
```

Tokens are long-lived; list them with `fokus-mcp token list` and revoke with
`fokus-mcp token revoke <id>`.

## Quick start — local CLI

```bash
npm install -g @fokus-app/mcp
fokus-mcp login
fokus-mcp workspace               # pick your default workspace
```

Then follow [docs/openclaw.md](docs/openclaw.md) or [docs/hermes.md](docs/hermes.md).

## Requirements & limits

- Node.js 20+ where the agent runs; network access to the Fokus API (cloud or self-hosted
  via `FOKUS_API_URL` / `fokus-mcp login --api-url`).
- Local CLI login is interactive and must be done by a human once; credentials expire after
  ~7 days without use.
- For unattended/headless use, mint a long-lived token with `fokus-mcp token create` and
  connect via the hosted MCP endpoint — those tokens don't expire from idleness and are
  revoked explicitly (`fokus-mcp token revoke <id>`).

## License

MIT
