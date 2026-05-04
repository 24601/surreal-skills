# SurrealMCP -- Model Context Protocol Server for SurrealDB

SurrealMCP is the official Model Context Protocol (MCP) server published by
SurrealDB Labs. It exposes SurrealDB connections, queries, schema introspection,
and graph traversal as MCP tools so any MCP-compatible LLM client (Claude
Desktop, Claude Code, Cursor, Windsurf, OpenCode, Amp, Codex, Continue, etc.)
can read and write a SurrealDB instance through a single declarative
configuration entry instead of bespoke per-agent glue code.

> **Scope**: this rule covers the MCP server side. For host-side MCP client
> configuration (Claude Code's `~/.claude/mcp.json`, Cursor's
> `.cursor/mcp.json`, etc.) consult the host documentation. SurrealMCP is the
> *server* that those clients connect to.

---

## When to Use SurrealMCP

| Use case | Why MCP fits |
|----------|--------------|
| Long-running agent loops over a project DB | One-shot connection setup, cached schema, structured tool surface |
| Coding agents that need to introspect the live schema before writing SurrealQL | `schema.introspect` returns the same shape every call, no hand-rolled `INFO FOR DB` parsing |
| Multi-host agent deployments (Claude + Cursor + Codex on the same DB) | Single MCP config, identical tools across clients |
| Unattended automation (autoplan, /loop, scheduled agents) | Tool calls fail loudly; raw SDK calls get swallowed by streaming output |

If you only need ad-hoc SurrealQL execution from a CLI shell, the `surreal sql`
REPL is faster. Reach for SurrealMCP when an agent needs *programmatic* access
that survives across turns.

---

## Installation

The reference server is distributed as a single binary plus a `npx`-installable
launcher that wraps it for MCP host configs.

```bash
# Install via Cargo (preferred for offline / vendored installs)
cargo install surrealmcp

# Or via npm wrapper (auto-fetches the matching binary on first run)
npm install -g @surrealdb/surrealmcp

# Verify
surrealmcp --version
```

Containerized hosts can use the published image:

```bash
docker pull surrealdb/surrealmcp:latest
docker run --rm -i surrealdb/surrealmcp \
  --endpoint $SURREAL_ENDPOINT --user $SURREAL_USER --pass $SURREAL_PASS
```

> **Security note**: prefer auditable installs through Cargo, npm, or Docker
> over piping a shell installer. The Cargo and npm distributions both publish
> SHA-256 digests in their respective registries.

---

## Transports

SurrealMCP speaks the two transports defined by the MCP spec:

| Transport | When to pick it | Configuration |
|-----------|-----------------|---------------|
| **stdio** (default) | Local agents (Claude Desktop/Code, Cursor, Codex CLI) | Host launches the binary; MCP messages flow over stdin/stdout |
| **SSE / Streamable HTTP** | Remote agents, sandboxed environments, multi-tenant deployments | Bind a port; clients connect over HTTPS |

```bash
# stdio (host launches this command)
surrealmcp \
  --endpoint ws://localhost:8000/rpc \
  --user $SURREAL_USER --pass $SURREAL_PASS \
  --namespace test --database test

# Streamable HTTP (remote clients)
surrealmcp serve \
  --transport http --bind 127.0.0.1:8765 \
  --endpoint ws://localhost:8000/rpc \
  --user $SURREAL_USER --pass $SURREAL_PASS \
  --namespace test --database test
```

For HTTP mode behind a public network, terminate TLS at a reverse proxy
(Caddy, nginx, Traefik) and authenticate clients with bearer tokens via the
`--auth-token` flag. Never expose stdio mode through `socat` to the public
internet -- there is no transport-level auth on stdio.

---

## Configuring MCP Hosts

### Claude Code / Claude Desktop

Add the server to `~/.claude/mcp.json` (Claude Code) or
`~/Library/Application Support/Claude/claude_desktop_config.json` (Claude
Desktop):

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

`SURREAL_USER` / `SURREAL_PASS` are read from the env block; the binary will
exit with a clear error if they are unset.

### Cursor

`.cursor/mcp.json` uses the same shape:

```json
{
  "mcpServers": {
    "surrealdb": {
      "command": "surrealmcp",
      "args": ["--endpoint", "ws://localhost:8000/rpc"],
      "env": { "SURREAL_USER": "root", "SURREAL_PASS": "root" }
    }
  }
}
```

### Codex CLI

`~/.codex/mcp.json`:

```json
{
  "servers": {
    "surrealdb": {
      "command": "surrealmcp",
      "args": ["--endpoint", "ws://localhost:8000/rpc",
               "--namespace", "test", "--database", "test"]
    }
  }
}
```

### OpenCode / Amp / Continue

All three follow the standard MCP host config format above. Refer to each
host's docs for the exact config path; the inner `mcpServers` block is portable.

---

## Exposed Tools

SurrealMCP advertises the following tool catalog over `tools/list`. Tool names
use dotted namespaces; arguments are JSON-Schema validated server-side.

| Tool | Purpose | Mutating |
|------|---------|----------|
| `query` | Run a parameterized SurrealQL statement; returns one result block per statement | Depends on statement |
| `select` | Typed CRUD select for a record ID or table | No |
| `create` | Create a record with content | Yes |
| `update` | Replace a record's contents | Yes |
| `merge` | Partial update | Yes |
| `delete` | Delete a record or table | Yes |
| `relate` | Create a graph edge with optional content | Yes |
| `live` | Subscribe to a LIVE SELECT; events stream back as MCP notifications | No |
| `kill` | Cancel a running live subscription | No |
| `schema.introspect` | Full DB schema dump (tables, fields, indexes, events, accesses) | No |
| `schema.tables` | Table list with field/index counts | No |
| `schema.table` | Single-table detail | No |
| `info.db` | Wrap `INFO FOR DB` | No |
| `info.ns` | Wrap `INFO FOR NS` | No |
| `use` | Switch the active namespace and database for the session | No |
| `signin` | Re-authenticate within an existing session | No |

Read-only tools are safe to expose to autonomous loops. Mutating tools should
be gated behind host-level approval (Claude Code's permission prompts, Cursor's
trust dialog) when running against shared or production endpoints.

---

## Tool-Use Patterns

### Schema-Aware Code Generation

A coding agent should call `schema.introspect` before writing new SurrealQL so
it generates statements that match the live schema, not the LLM's prior beliefs:

```
1. tools/call schema.introspect
2. (agent reasons over returned tables/fields)
3. tools/call query with the parameterized statement
```

This pattern prevents the most common class of agent failure -- generating
queries against fields that no longer exist or that were renamed.

### Streaming Results

For large `SELECT *` reads, prefer the streaming `query` tool with the
`stream: true` argument over a single `select`. The MCP host receives a
sequence of `tools/progress` notifications and can render partial results
without buffering the full payload.

### Live Queries Through MCP

```
1. tools/call live { table: "user" }
   -> returns { subscriptionId: "lq_abc123" }
2. server pushes notifications/tools/progress with each CREATE/UPDATE/DELETE
3. tools/call kill { subscriptionId: "lq_abc123" } when done
```

Always pair `live` with a matching `kill`; orphaned subscriptions accumulate on
the SurrealDB side until the WebSocket disconnects.

---

## Authentication

SurrealMCP supports the full SurrealDB auth surface:

```bash
# Root credentials (LOCAL DEV ONLY)
surrealmcp --user root --pass root

# Namespace credentials
surrealmcp --user ns_user --pass ... --namespace my_ns

# Database credentials
surrealmcp --user db_user --pass ... --namespace my_ns --database my_db

# Record-level access (DEFINE ACCESS user_access)
surrealmcp --access user_access --variables 'email=user@example.com,password=...' \
  --namespace my_ns --database my_db

# JWT bearer token (already issued)
surrealmcp --token "$JWT"
```

For host configs, prefer environment variables (`SURREAL_USER`,
`SURREAL_PASS`, `SURREAL_TOKEN`) over passing secrets on the command line --
the latter shows up in `ps` output.

---

## Production Deployment

| Concern | Recommendation |
|---------|----------------|
| Multi-tenant exposure | Run one SurrealMCP process per tenant / namespace; isolate with systemd or Docker |
| Network exposure | Streamable HTTP behind TLS + bearer token auth; never expose stdio |
| Rate limiting | Use the `--max-concurrent-tools <N>` flag (default 16) to cap per-session load |
| Audit logging | `--log-format json` produces structured logs to stderr; ship to your log pipeline |
| Health checks | `surrealmcp ping` exits 0 if the underlying SurrealDB is reachable |
| Permission posture | Always pair with scoped DB users (DEFINE USER ... ON DATABASE ROLES VIEWER) -- do not run MCP as root in production |

---

## Comparison With Direct SDK Use

| Concern | SurrealMCP | Direct SDK (rules/sdks.md) |
|---------|------------|----------------------------|
| Multi-host portability | One config, all hosts | Each agent integrates separately |
| Schema visibility | First-class `schema.*` tools | Hand-rolled `INFO FOR DB` parsing |
| Live queries through agent | First-class `live` tool, MCP notifications | SDK live API, agent must marshal events |
| Custom logic in client | Limited to MCP tool surface | Full SDK API, query builder, codecs |
| Performance | Adds one process hop | Direct connection |
| Fit for app code | No | Yes |
| Fit for agent integration | Yes | Possible but more code |

Use SurrealMCP when an agent is the consumer; use the SDK when application
code is.

---

## Cross-References

- `rules/sdks.md` -- direct SDK usage for application code
- `rules/security.md` -- DEFINE USER + DEFINE ACCESS to scope the credentials SurrealMCP runs as
- `rules/editor-tooling.md` -- editor-side LSP/extension tooling (complements MCP servers)
- `rules/deployment.md` -- production hardening posture for the SurrealDB instance MCP connects to
- `skills/surrealmcp/SKILL.md` -- compact MCP-specific sub-skill for host-side install snippets
