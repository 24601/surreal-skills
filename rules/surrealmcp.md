# SurrealMCP -- Model Context Protocol Server for SurrealDB

> **v1.4.1 status note:** the v1.4.0 version of this rule documented an
> install path, CLI shape, env-var names, and tool catalog that did not
> match the upstream `surrealdb/surrealmcp` repository. This file has been
> shrunk to verified content only; for anything not covered here, treat
> the upstream README as the source of truth and pin work to a specific
> SurrealMCP commit. Detail expansion is deferred to v1.5.0 after a full
> upstream verification pass.

SurrealMCP is the official Model Context Protocol (MCP) server published
under `surrealdb/surrealmcp`. It exposes a SurrealDB connection (local or
SurrealDB Cloud) to MCP-compatible clients (Claude Desktop, Cursor,
GitHub Copilot in VS Code, Zed, n8n, etc.) so an agent can read and
write a SurrealDB instance without bespoke per-host glue.

- Upstream: `https://github.com/surrealdb/surrealmcp`
- Status: preview
- Source-of-truth README: pin to upstream HEAD; consult for every
  install/CLI/tool detail not duplicated here

---

## When to Use SurrealMCP

| Use case | Why MCP fits |
|----------|--------------|
| Long-running agent loops over a project DB | One-shot connection setup, structured tool surface |
| Multi-host agent deployments (Claude + Cursor + Copilot on the same DB) | Single MCP config, identical tools across clients |
| Unattended automation (autoplan, /loop, scheduled agents) | Tool calls fail loudly; raw SDK calls get swallowed by streaming output |

Reach for SurrealMCP when an agent needs *programmatic* access that
survives across turns. Reach for the SDK (`rules/sdks.md`) when
application code is the consumer, not an agent.

---

## Installation (verified)

The upstream README documents two install paths:

```bash
# From source
git clone https://github.com/surrealdb/surrealmcp
cd surrealmcp
cargo install --path .

# Docker (preferred for unattended hosts)
docker run --rm -i --pull always surrealdb/surrealmcp:latest start
```

`cargo install surrealmcp` does **not** work -- the crate is not
published to crates.io. There is no published `npm` package either.
Use the source or Docker paths only.

---

## CLI Shape (verified)

The binary requires the `start` subcommand for every mode:

```bash
# stdio (default; what most MCP hosts launch)
surrealmcp start

# HTTP server
surrealmcp start --bind-address 127.0.0.1:8000

# Unix socket
surrealmcp start --socket-path /tmp/surrealmcp.sock
```

Connection flags (verified against upstream README):

| Flag | Purpose |
|------|---------|
| `--endpoint <URL>` | SurrealDB endpoint (e.g. `ws://localhost:8000/rpc`) |
| `--ns <name>` | Namespace |
| `--db <name>` | Database |
| `--user <name>` / `--pass <secret>` | DB credentials |
| `--bind-address <host:port>` | HTTP bind |
| `--server-url <url>` | Public-facing server URL |
| `--cloud-auth-server <url>` | SurrealDB Cloud auth server |
| `--expected-audience <aud>` | JWT audience validation |
| `--rate-limit-rps <n>` / `--rate-limit-burst <n>` | Rate limiting |
| `--auth-disabled` | Disable auth (dev only) |

Health check is HTTP, not a CLI subcommand:

```bash
curl http://localhost:8000/health
```

There is no `surrealmcp ping`, `surrealmcp serve`, `--transport`,
`--bind` (use `--bind-address`), `--auth-token`, `--max-concurrent-tools`,
`--namespace` (use `--ns`), `--database` (use `--db`), or
`--log-format json` flag in current upstream. Logging is configured via
`RUST_LOG` env var (`tracing-subscriber`).

---

## Environment Variables (verified)

```bash
# Connection
SURREALDB_URL=ws://localhost:8000/rpc
SURREALDB_NS=mynamespace
SURREALDB_DB=mydatabase
SURREALDB_USER=root
SURREALDB_PASS=root

# Server (HTTP mode)
SURREAL_MCP_BIND_ADDRESS=127.0.0.1:8000
SURREAL_MCP_SERVER_URL=https://mcp.surrealdb.com
SURREAL_CLOUD_AUTH_SERVER=https://auth.surrealdb.com
SURREAL_MCP_EXPECTED_AUDIENCE=https://custom.audience.com/
SURREAL_MCP_RATE_LIMIT_RPS=100
SURREAL_MCP_RATE_LIMIT_BURST=200
SURREAL_MCP_AUTH_REQUIRED=false
SURREAL_MCP_CLOUD_ACCESS_TOKEN=...
SURREAL_MCP_CLOUD_REFRESH_TOKEN=...
```

Note the `SURREALDB_*` prefix for connection vars and the
`SURREAL_MCP_*` prefix for server-side configuration. The
surreal-skills-wide `SURREAL_USER` / `SURREAL_PASS` convention used
in other rules is **not** what surrealmcp reads -- map your env
explicitly when wiring host configs.

---

## Configuring MCP Hosts

The upstream README includes installation walk-throughs for Cursor,
Claude Desktop, GitHub Copilot in VS Code, Zed, and n8n. Use those as
the source-of-truth for each host's exact JSON path and key shape; the
example below is the common skeleton.

```json
{
  "mcpServers": {
    "SurrealDB": {
      "command": "docker",
      "args": [
        "run", "--rm", "-i", "--pull", "always",
        "surrealdb/surrealmcp:latest",
        "start"
      ]
    }
  }
}
```

For a non-Docker invocation, swap to `"command": "surrealmcp"` with
`"args": ["start"]` and pass connection details via the `env` block
using the verified env-var names above.

> **Cross-host config drift:** different hosts use different top-level
> keys (`mcpServers` vs `servers` vs `mcp.servers`). Always consult the
> host's own MCP docs for the exact path and shape.

---

## Available Tools (verified)

The upstream README groups the tools as follows. Tool wire-names are
snake_case (verified against `src/tools/mod.rs`); the names below are
the prose form from the README.

**Database operations:** Query, Select, Insert, Create, Upsert,
Update, Delete, Relate.

**Connection management:** Connect Endpoint, Use Namespace,
Use Database, List Namespaces, List Databases, Disconnect Endpoint.

**Cloud operations:** List Cloud Organizations, List Cloud Instances,
Create Cloud Instance, Pause / Resume Cloud Instance, Get Cloud
Instance Status.

The `connect_endpoint` tool accepts a target like
`memory`, `file:/path`, `rocksdb:/path`, `ws://host:port`,
`http://host:port`, or `cloud:<instance_id>`.

> **Pending verification:** exact tool-argument shapes, JSON-Schema
> definitions, and the precise list of tools at any given upstream tag.
> Use `tools/list` against a running server and pin to a specific
> upstream commit.

---

## Production Posture

| Concern | Posture |
|---------|---------|
| Network exposure | HTTP behind TLS termination + auth required; never expose stdio over a TCP forwarder |
| Auth | Bearer JWT with JWKS validation against the configured auth server; `--auth-disabled` is dev-only |
| Rate limiting | `--rate-limit-rps` / `--rate-limit-burst` flags |
| Health | `GET /health` endpoint |
| Audit logging | Configure `RUST_LOG`; SurrealMCP uses `tracing-subscriber` |
| Permission posture | Run with scoped DB credentials (`DEFINE USER ... ON DATABASE ROLES VIEWER`); see `rules/security.md` |

---

## Cross-References

- `rules/sdks.md` -- direct SDK use for application code
- `rules/security.md` -- DEFINE USER / DEFINE ACCESS for scoped credentials
- `rules/editor-tooling.md` -- editor LSP tooling (complements MCP)
- `rules/deployment.md` -- production hardening for the SurrealDB instance
- Upstream README: `https://github.com/surrealdb/surrealmcp/blob/main/README.md`
- Source-of-truth docs: `https://surrealdb.com/docs/integrations/`
