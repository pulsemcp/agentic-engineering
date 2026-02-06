# Agentic Engineering

## Principles

Agentic coding is often only as good as the feedback loops it can use to inspect and verify completion of work. 

### Verification in an agentic loop: assertion and observability

There are two oft-missing components to a strong agentic loop:

1. **Assertion** — the agent proves its work is correct. This means running tests, type checkers, linters, build steps, and any other pass/fail gate before moving on. The goal is a tight loop: write code, run checks, fix failures, repeat — all without human intervention.

2. **Observability** — the agent can see what's actually happening at runtime - you empower it with debugging capabilities. Logs, error traces, deployment status, metrics — when an agent has access to this data, it can diagnose the real problem instead of guessing.

### MCP's role: connecting and constraining

MCP is the standard interface for giving agents access to tools and data. It matters here for two reasons:

**Connecting.** MCP servers plug agents into your real infrastructure — databases, cloud APIs, CI/CD pipelines, monitoring systems — through a standardized protocol. Instead of fragile bash scripts and ad-hoc tool wrappers, you get structured tool definitions that are easy to reason about and designed for an LLM-forward era. This is what makes verification loops practical: the agent can call `create_database_index`, `check_deployment_status`, or `query_logs` as it identifies and fixes problems and builds net new functionality.

**Constraining.** MCP configurations let you control exactly what an agent can do in a given session. You can scope a server to read-only mode, filter which tools are exposed, limit access to specific resources, and template secrets through environment variables so credentials never touch the context window. This is how you make agents safe enough to run autonomously — not by hoping the model behaves, but by making dangerous actions structurally impossible in that session's configuration.

With this combination of connecting and constraining agents can iterate their way to completion, and you constrain their access so the blast radius of mistakes is bounded.

## Running Claude Code

Create a `.env` file at the project root (it's gitignored) with your secrets, then use this to start a session:

```bash
set -a && source .env && set +a && ENABLE_TOOL_SEARCH=false claude --dangerously-skip-permissions
```

**Why `set -a && source .env && set +a`?**

MCP server configurations in `mcp-servers/mcp.json` use `${VAR}` interpolation for secrets (API keys, tokens, AWS credentials, etc.). These variables must be present in your shell environment before launching Claude Code so they're available to child processes (i.e., MCP servers). `set -a` / `set +a` tells the shell to automatically export every variable that gets sourced. See `mcp-servers/SETUP.md` for what credentials each server needs and how to obtain them.

**Why `ENABLE_TOOL_SEARCH=false`?**

MCP works best when you [pre-emptively configure](https://www.pulsemcp.com/posts/agentic-mcp-configuration) what MCP servers should be active in a session. It _is_ more up front work than simply installing all your servers and ignoring the setup on a session-per-session basis, but we believe it's worth building some personal tooling around that dynamic rather than hoping automated "search" gets it right on the fly. This makes building reliable workflows and debugging issues easier to grok and reason through.

**Why `--dangerously-skip-permissions`?** To effectively agentically engineer, you don't want to be pinged for permissions approvals any more than is absolutely necessary. `--dangerously-skip-permissions` makes this the default, albeit with obvious associated risks. An alternative if you are working in a sensitive environment: slowly build up an always-allow-list of commands that get checked into `.claude/settings.json`.

## Useful skills

This repo ships Claude Code skills for managing a personal catalog of MCP servers — from discovery through to session configuration.

- **`/create-server-json`** — Research an MCP server (from a URL, name, or repo) and produce a validated `server.json` definition. Downloads the official schema and validates the output with `ajv` before saving. Results go into `mcp-servers/servers.json`.

- **`/create-mcp-json`** — Transform a `server.json` into a ready-to-use client configuration. Writes actionable descriptions, templates all env vars as `${ENV_VAR_NAME}` placeholders, and validates against `docs/mcp.schema.json`. Results go into `mcp-servers/mcp.json`.

- **`/adjust-claude-mcp-json`** — Pick servers from `mcp-servers/mcp.json` and generate (or update) a `.mcp.json` for a specific project. Use this to activate a subset of your catalog for a Claude Code session.
