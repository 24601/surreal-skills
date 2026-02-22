---
name: surrealfs
description: "SurrealFS virtual filesystem for AI agents. Rust core + Python agent (Pydantic AI). Persistent file operations backed by SurrealDB. Part of the surreal-skills collection."
license: MIT
metadata:
  version: "1.0.2"
  author: "24601"
  parent_skill: "surrealdb"
  snapshot_date: "2026-02-19"
  upstream:
    repo: "surrealdb/surrealfs"
    sha: "0008a3a94dbe"
---

# SurrealFS -- Virtual Filesystem for AI Agents

SurrealFS provides a persistent, queryable virtual filesystem backed by SurrealDB.
Designed for AI agents that need durable file operations, hierarchical storage,
and content search across sessions.

## Components

| Component | Crate/Package | Language | Purpose |
|-----------|---------------|----------|---------|
| Core Library | `surrealfs` | Rust | Filesystem operations, CLI REPL, SurrealDB storage layer |
| AI Agent | `surrealfs-ai` | Python (Pydantic AI) | Agent interface with tool integration, HTTP hosting |

## Rust Core -- Commands

The `surrealfs` crate provides a REPL with POSIX-like commands:

| Command | Description |
|---------|-------------|
| `ls` | List directory contents |
| `cat` | Display file contents |
| `tail` | Show last lines of a file |
| `nl` | Number lines of a file |
| `grep` | Search file contents |
| `touch` | Create empty file |
| `mkdir` | Create directory |
| `write_file` | Write content to file |
| `edit` | Edit file contents |
| `cp` | Copy file |
| `cd` | Change directory |
| `pwd` | Print working directory |

Supports piping from external commands: `curl https://example.com > /pages/example.html`

Storage backends:
- Embedded RocksDB (local)
- Remote SurrealDB via WebSocket

## Python AI Agent

Built on Pydantic AI with tools that mirror the filesystem commands.

```python
from surrealfs_ai import build_chat_agent

# Create the agent (default LLM: Claude Haiku)
agent = build_chat_agent()

# Expose over HTTP
import uvicorn
app = agent.to_web()
uvicorn.run(app, host="127.0.0.1", port=7932)
```

Features:
- Default LLM: Claude Haiku
- Telemetry via Pydantic Logfire (OpenTelemetry)
- All filesystem operations available as agent tools
- HTTP hosting (default port 7932)
- Path normalization (cannot escape `/`)

## Quick Start

```bash
# Install the Rust core
cargo install surrealfs

# Start the REPL with embedded storage
surrealfs

# Or connect to a remote SurrealDB instance
surrealfs --endpoint ws://localhost:8000 --user root --pass root --ns agent --db workspace

# Install the Python agent
pip install surrealfs-ai

# Run the agent HTTP server
python -m surrealfs_ai --host 127.0.0.1 --port 7932
```

## Use Cases

- Persistent workspace for AI agent sessions
- Hierarchical document storage with metadata queries
- Multi-agent shared file access with SurrealDB permissions
- Content strategy and knowledge management
- Project scaffolding and template management

## Full Documentation

See the main skill's rule file for complete guidance:
- **[rules/surrealfs.md](../../rules/surrealfs.md)** -- architecture, Rust core API, Python agent setup, SurrealDB schema, multi-agent patterns, and deployment
- **[surrealdb/surrealfs](https://github.com/surrealdb/surrealfs)** -- upstream repository
