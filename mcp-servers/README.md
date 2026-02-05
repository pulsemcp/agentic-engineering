# mcp-servers

A curated set of trusted MCP servers and their configurations.

## Files

### `servers.json`

A collection of `server.json` entries for each trusted server. These follow the [MCP Registry server.json specification](https://github.com/modelcontextprotocol/registry/blob/main/docs/reference/server-json/generic-server-json.md) and describe **what each server is and how it can be run** — identity, packages, remotes, environment variables, and other metadata.

Think of this as the catalog: "here are the servers we trust, and here's everything about them."

### `mcp.json`

Specific, useful configurations derived from the server.json entries above. Each entry is a fully-resolved client configuration ready to be used by an MCP client (Claude Code, Claude Desktop, VS Code, etc.).

Think of this as the playbook: "here's exactly how we connect to each server, with the settings we actually use."

A single server in `servers.json` may have multiple corresponding entries in `mcp.json` — for example, one configuration for full read/write access and another for read-only use.

## Relationship between the two

```
servers.json                          mcp.json
┌─────────────────────────┐          ┌──────────────────────────────┐
│ server.json entries      │          │ client configs               │
│                          │          │                              │
│ "what can be configured" │  ──────► │ "how we actually connect"    │
│                          │          │                              │
│ - packages, remotes      │          │ - command, args, env         │
│ - env var specs          │          │ - url, headers               │
│ - transport options      │          │ - concrete secret references │
└─────────────────────────┘          └──────────────────────────────┘
```

## Adding a server

Use the `/create-server-json` skill to generate a server.json entry, then `/create-mcp-json` to produce the corresponding client configuration.
