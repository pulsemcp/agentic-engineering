# Agentic Engineering

## Recommendations

Use this when running Claude Code:

```bash
ENABLE_TOOL_SEARCH=false claude --dangerously-skip-permissions
```

Or do this in `.claude/settings.json` for the `ENABLE_TOOL_SEARCH=false` bit:

```
```

**Why `ENABLE_TOOL_SEARCH=false`?**

MCP works best when you [pre-emptively configure](https://www.pulsemcp.com/posts/agentic-mcp-configuration) what MCP servers should be active in a session. It _is_ more up front work than simply installing all your servers and ignoring the setup on a session-per-session basis, but we believe it's worth building some personal tooling around that dynamic rather than hoping automated "search" gets it right on the fly. This makes building reliable workflows and debugging issues easier to grok and reason through.

**Why `--dangerously-skip-permissions`?** To effectively agentically engineer, you don't want to be pinged for permissions approvals any more than is absolutely necessary. `--dangerously-skip-permissions` makes this the default, albeit with obvious associated risks. An alternative if you are working in a sensitive environment: slowly build up an always-allow-list of commands that get checked into `.claude/settings.json`.