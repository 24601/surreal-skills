---
name: surrealmcp
description: "SurrealMCP -- Model Context Protocol server for SurrealDB. Lets MCP-compatible LLM hosts (Claude Code/Desktop, Cursor, Codex, OpenCode, Amp, Continue, Windsurf) read and write a SurrealDB instance through a single config entry. Part of the surreal-skills collection."
license: MIT
metadata:
  version: "1.4.0"
  author: "24601"
  parent_skill: "surrealdb"
  snapshot_date: "2026-05-03"
  upstream:
    repo: "surrealdb/surrealmcp"
    release: "v0.2.0"
---

# SurrealMCP -- MCP Server for SurrealDB

SurrealMCP exposes SurrealDB as a Model Context Protocol server so any MCP host
can query, mutate, introspect, and live-subscribe over one declarative config
entry instead of bespoke per-agent integration code.

## Quick Start

```bash
# Install (Cargo)
cargo install surrealmcp

# Or via npm wrapper
npm install -g @surrealdb/surrealmcp

# Run as stdio server (the host launches this command)
surrealmcp \
  --endpoint ws://localhost:8000/rpc \
  --user $SURREAL_USER --pass $SURREAL_PASS \
  --namespace test --database test
```

## Host Config Snippet

Claude Code (`~/.claude/mcp.json`), Cursor (`.cursor/mcp.json`), Codex
(`~/.codex/mcp.json`), OpenCode, Amp, Continue all share the same shape:

```json
{
  "mcpServers": {
    "surrealdb": {
      "command": "surrealmcp",
      "args": [
        "--endpoint", "ws://localhost:8000/rpc",
        "--namespace", "test",
        "--database", "test"
      ],
      "env": {
        "SURREAL_USER": "root",
        "SURREAL_PASS": "root"
      }
    }
  }
}
```

## Exposed Tools

| Tool | Purpose |
|------|---------|
| `query` | Parameterized SurrealQL execution |
| `select` / `create` / `update` / `merge` / `delete` | Typed CRUD |
| `relate` | Graph edge creation |
| `live` / `kill` | LIVE SELECT subscriptions over MCP notifications |
| `schema.introspect` / `schema.tables` / `schema.table` | Schema introspection |
| `info.db` / `info.ns` | INFO statement wrappers |
| `use` / `signin` | Session-level namespace + auth changes |

Read-only tools are safe for autonomous loops. Mutating tools should be gated
through the host's permission prompt against shared / production DBs.

## Production Notes

- Prefer Streamable HTTP transport behind TLS + bearer token for remote hosts;
  never expose stdio publicly.
- Run MCP as a scoped DB user (`DEFINE USER ... ON DATABASE`), not root.
- Use `--log-format json` for structured stderr suitable for log shipping.
- `surrealmcp ping` is the canonical health check.

## Full Documentation

- **[rules/surrealmcp.md](../../rules/surrealmcp.md)** -- transports, full tool catalog, host configs, auth, deployment, security, comparison with direct SDK use
- **[surrealdb/surrealmcp](https://github.com/surrealdb/surrealmcp)** -- upstream repository
- **[modelcontextprotocol.io](https://modelcontextprotocol.io)** -- MCP specification
