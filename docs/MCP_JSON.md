# mcp.json Specification

> **Warning:** This is not yet an official standard. There is active interest within the MCP community in standardizing a client-side configuration format like this, and it is likely to become one before long. Until then, treat this specification as a practical, opinionated proposal that may evolve as standardization efforts progress.

This document defines the `mcp.json` configuration format for MCP (Model Context Protocol) servers. This specification provides a standardized format for client-side MCP server configurations.

## Scope

This specification is intended for:
- Internal tools that need to manage MCP server configurations programmatically
- Applications that instantiate MCP servers at runtime (like agent orchestrators)
- Reference documentation for the structure of MCP server entries

**Note:** This format differs from Claude Code's native `.mcp.json` format, which wraps server configurations in a `mcpServers` object. See [Relationship to Claude Code](#relationship-to-claude-code) for details.

## Overview

An `mcp.json` file defines a collection of MCP servers that an MCP client application can connect to. Each server is identified by a unique name key and configured with transport-specific settings.

## Relationship to server.json

The MCP ecosystem has two related but distinct configuration formats:

| Format | Purpose | Configurability |
|--------|---------|-----------------|
| `server.json` | Server package specification for registries | Highly configurable with variables, templates, and user-adjustable parameters |
| `mcp.json` | Client-side server configuration | Fully resolved; only auth secrets are interpolated |

**server.json** (defined by the [MCP Registry](https://github.com/modelcontextprotocol/registry)) is a package specification format used to describe MCP server packages for discovery and distribution. It supports:
- Multiple package types (npm, PyPI, Docker, etc.)
- Metadata including version, description, and icons
- Runtime hints (npx, docker, python, etc.)
- Environment variable specifications with `isRequired` and `isSecret` flags

**mcp.json** represents a fully-formed, opinionated configuration for a specific MCP client. Each entry in an `mcp.json` is what you get *after* resolving a `server.json` template with concrete values. The only remaining configurability is authentication secrets via environment variable interpolation.

Think of it this way:
- `server.json` = "Here's how this server *can* be configured"
- `mcp.json` = "Here's exactly how this client *will* connect to its servers"

## Relationship to Claude Code

Claude Code uses a similar but distinct format for its `.mcp.json` files:

| This Specification | Claude Code Format |
|-------------------|-------------------|
| Servers at root level | Servers nested under `mcpServers` key |
| `${VAR}` for runtime interpolation | Various patterns depending on tooling |
| `type` field required | `type` field optional (defaults to stdio) |

**Claude Code format:**
```json
{
  "mcpServers": {
    "server-name": {
      "command": "npx",
      "args": ["..."]
    }
  }
}
```

**This specification:**
```json
{
  "server-name": {
    "type": "stdio",
    "command": "npx",
    "args": ["..."]
  }
}
```

This project's `mcp-json-profiles/` directory uses the Claude Code format. See [MCP.md](./MCP.md) and [mcp-json-profiles/README.md](../mcp-json-profiles/README.md) for profile-based configuration.

## Schema

The formal JSON schema is available at [`mcp.schema.json`](./mcp.schema.json).

## File Structure

```json
{
  "server-name": {
    "title": "Human-Readable Name",
    "type": "stdio",
    "command": "executable",
    "args": ["arg1", "arg2"],
    "env": {
      "VAR_NAME": "value"
    }
  }
}
```

The root object is a map of server names to server configurations. Server names must be alphanumeric with hyphens, underscores, or brackets allowed (matching the pattern `^[a-zA-Z0-9_\[\]-]+$`).

## Server Configuration

### Common Properties

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `title` | string | No | Human-readable display name (1-100 chars) |
| `description` | string | No | Human-readable description of what the server provides (max 500 chars) |
| `type` | string | Yes | Transport type: `stdio`, `sse`, or `streamable-http` |

### Transport Types

#### stdio (Local Process)

For servers that run as local processes and communicate via stdin/stdout.

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `command` | string | Yes | Executable command to run |
| `args` | string[] | No | Command-line arguments |
| `env` | object | No | Environment variables for the process |

Example:

```json
{
  "filesystem": {
    "title": "Filesystem Server",
    "description": "Provides file system access within specified directories",
    "type": "stdio",
    "command": "npx",
    "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/allowed/directory"],
    "env": {
      "NODE_ENV": "production"
    }
  }
}
```

#### sse (Server-Sent Events)

For remote servers using the SSE transport protocol.

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `url` | string | Yes | Server endpoint URL |
| `headers` | object | No | HTTP headers for requests |

Example:

```json
{
  "remote-tools": {
    "title": "Remote Tools Server",
    "type": "sse",
    "url": "https://mcp.example.com/sse",
    "headers": {
      "Authorization": "Bearer ${API_TOKEN}"
    }
  }
}
```

#### streamable-http

For remote servers using HTTP streaming.

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `url` | string | Yes | Server endpoint URL |
| `headers` | object | No | HTTP headers for requests |

Example:

```json
{
  "api-server": {
    "title": "API Server",
    "type": "streamable-http",
    "url": "https://api.example.com/mcp",
    "headers": {
      "X-API-Key": "${API_KEY}"
    }
  }
}
```

## Environment Variable Interpolation

String values in `mcp.json` support environment variable interpolation for injecting secrets at runtime.

### Syntax

| Pattern | Description |
|---------|-------------|
| `${VAR}` | Replaced with the value of environment variable `VAR` |
| `${VAR:-default}` | Uses `VAR` if set, otherwise falls back to `default` |

### Supported Fields

Interpolation is supported in:
- `command`
- `args` (each element)
- `env` (values only)
- `url`
- `headers` (values only)

### Purpose and Scope

**Important:** Environment variable interpolation is primarily intended for authentication secrets. This includes:

- API keys (e.g., `${OPENAI_API_KEY}`)
- Bearer tokens (e.g., `Bearer ${AUTH_TOKEN}`)
- Other credentials required for server authentication

While interpolation technically works in other fields (like `url` or `command`), the `mcp.json` file is meant to be a fully-formed configuration. Using interpolation for non-secret values is discouraged as it undermines its purpose as a concrete client configuration.

### Examples

**API Key in environment:**

```json
{
  "github": {
    "title": "GitHub Tools",
    "type": "stdio",
    "command": "npx",
    "args": ["-y", "@modelcontextprotocol/server-github"],
    "env": {
      "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}"
    }
  }
}
```

**Bearer token in header:**

```json
{
  "authenticated-api": {
    "title": "Authenticated API",
    "type": "streamable-http",
    "url": "https://api.example.com/mcp",
    "headers": {
      "Authorization": "Bearer ${API_TOKEN}"
    }
  }
}
```

### OAuth-Based Servers

Servers using OAuth authentication handle authentication out-of-band. The OAuth flow is managed by the MCP client application, not through static configuration values.

MCP clients discover OAuth endpoints automatically via standard well-known metadata endpoints (RFC 8414 `/.well-known/oauth-authorization-server` or RFC 9728 `/.well-known/oauth-protected-resource`). No explicit OAuth configuration is needed in `mcp.json`.

```json
{
  "linear": {
    "title": "Linear",
    "description": "Linear project management integration",
    "type": "streamable-http",
    "url": "https://mcp.linear.app/mcp"
  }
}
```

**Client credentials** (`client_id`, `client_secret`) are **application-specific** (identify YOUR app to the OAuth provider). These belong in your application's configuration (e.g., environment variables, encrypted credentials), not in `mcp.json`.

## Complete Example

```json
{
  "filesystem": {
    "title": "Filesystem Access",
    "description": "Provides read/write access to the project directory",
    "type": "stdio",
    "command": "npx",
    "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/projects"]
  },
  "github": {
    "title": "GitHub Integration",
    "description": "GitHub API integration for repository management",
    "type": "stdio",
    "command": "npx",
    "args": ["-y", "@modelcontextprotocol/server-github"],
    "env": {
      "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}"
    }
  },
  "internal-api": {
    "title": "Internal API",
    "description": "Connection to internal services",
    "type": "streamable-http",
    "url": "https://internal.example.com/mcp",
    "headers": {
      "Authorization": "Bearer ${INTERNAL_API_KEY}",
      "X-Team-ID": "engineering"
    }
  },
  "linear": {
    "title": "Linear",
    "description": "Linear project management integration",
    "type": "streamable-http",
    "url": "https://mcp.linear.app/mcp"
  }
}
```

## Validation

Validate your configuration against the schema:

```bash
# Using ajv-cli (Node.js)
npm install -g ajv-cli
npx ajv-cli validate -s docs/mcp.schema.json -d your-config.json

# Using check-jsonschema (Python)
pip install check-jsonschema
check-jsonschema --schemafile docs/mcp.schema.json your-config.json
```

## Error Handling

- If a required environment variable (without a default) is not set, the MCP client should fail with a clear error message indicating which variable is missing.
- Invalid transport type configurations (e.g., `url` with `stdio`) should be rejected at parse time.
