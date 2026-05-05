---
name: surrealmcp
description: "SurrealMCP -- Model Context Protocol server for SurrealDB. Lets MCP-compatible LLM hosts (Claude Desktop, Cursor, GitHub Copilot in VS Code, Zed, n8n) read and write a SurrealDB instance through a single config entry instead of bespoke per-agent integration code. Part of the surreal-skills collection."
license: MIT
metadata:
  version: "1.4.5"
  author: "24601"
  parent_skill: "surrealdb"
  snapshot_date: "2026-05-05"
  upstream:
    repo: "surrealdb/surrealmcp"
    release: "v0.4.0"
---

# SurrealMCP -- MCP Server for SurrealDB

SurrealMCP exposes SurrealDB as a Model Context Protocol server so any
MCP host can query, mutate, introspect, and manage cloud instances
through one declarative config entry instead of bespoke per-agent
integration code.

> **v1.4.1 status note:** the v1.4.0 quick-start documented an
> install path, CLI shape, env vars, and tool catalog that did not
> match upstream. This file has been corrected against
> `surrealdb/surrealmcp/README.md`. For full detail see
> [rules/surrealmcp.md](../../rules/surrealmcp.md).

## Quick Start

`surrealmcp` is **not** published to crates.io or npm. Install from
source or use the official Docker image:

```bash
# From source
git clone https://github.com/surrealdb/surrealmcp
cd surrealmcp
cargo install --path .

# Docker (preferred)
docker run --rm -i --pull always surrealdb/surrealmcp:latest start

# Run as stdio server (the host launches this command)
surrealmcp start \
  --endpoint ws://localhost:8000/rpc \
  --ns test --db test \
  --user root --pass root
```

The `start` subcommand is mandatory; `surrealmcp serve` and bare
`surrealmcp` (no subcommand) are not valid invocations.

## Host Config Snippet

Different MCP hosts use different top-level keys -- always consult the
host's own MCP docs for the exact shape. The Docker invocation works
across most hosts that follow the `mcpServers` convention:

```json
{
  "mcpServers": {
    "SurrealDB": {
      "command": "docker",
      "args": [
        "run", "--rm", "-i", "--pull", "always",
        "surrealdb/surrealmcp:latest",
        "start"
      ],
      "env": {
        "SURREALDB_URL": "ws://localhost:8000/rpc",
        "SURREALDB_NS": "test",
        "SURREALDB_DB": "test",
        "SURREALDB_USER": "root",
        "SURREALDB_PASS": "root"
      }
    }
  }
}
```

Note the env var names: `SURREALDB_*` (not `SURREAL_*`). Server-side
tuning uses a separate `SURREAL_MCP_*` prefix.

## Exposed Tools

The upstream README groups tools as:

- **Database operations:** `query`, `select`, `insert`, `create`,
  `upsert`, `update`, `delete`, `relate`
- **Connection management:** `connect_endpoint`, `use_namespace`,
  `use_database`, `list_namespaces`, `list_databases`,
  `disconnect_endpoint`
- **Cloud operations:** `list_cloud_organizations`,
  `list_cloud_instances`, `create_cloud_instance`, `pause_cloud_instance`,
  `resume_cloud_instance`, `get_cloud_instance_status`

Tool wire-names are snake_case. Read-only tools are safe for
autonomous loops; mutating tools should be gated through the host's
permission prompt against shared / production DBs.

## Production Notes

- HTTP mode behind TLS + bearer JWT (validated via JWKS) for remote
  hosts; never expose stdio publicly.
- Run MCP as a scoped DB user (`DEFINE USER ... ON DATABASE`), not root.
- Health check is HTTP: `curl http://localhost:8000/health`.
- Configure logging via `RUST_LOG` (the binary uses `tracing-subscriber`).
- Rate limit with `--rate-limit-rps` / `--rate-limit-burst`.

## Full Documentation

- **[rules/surrealmcp.md](../../rules/surrealmcp.md)** -- verified
  install / CLI / env-var / tool catalog detail
- **[surrealdb/surrealmcp](https://github.com/surrealdb/surrealmcp)** --
  upstream repository (source-of-truth README)
- **[modelcontextprotocol.io](https://modelcontextprotocol.io)** --
  MCP specification
