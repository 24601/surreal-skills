---
name: surrealfs
description: "SurrealFS virtual filesystem for AI agents. Rust core + Python agent (Pydantic AI). Persistent file operations backed by SurrealDB. Part of the surreal-skills collection."
license: MIT
metadata:
  version: "1.0.0"
  author: "24601"
  parent_skill: "surrealdb"
---

# SurrealFS -- Virtual Filesystem for AI Agents

SurrealFS provides a persistent, queryable virtual filesystem backed by SurrealDB. It is designed for AI agents that need durable file operations, metadata tracking, and content search across agent sessions.

## Components

| Component | Language | Purpose |
|-----------|----------|---------|
| Core Library | Rust | Low-level filesystem operations, SurrealDB storage layer |
| Python Agent | Python (Pydantic AI) | High-level agent interface, tool integration |

## Quick Start

```python
from surrealfs import SurrealFSAgent

# Initialize the filesystem agent
agent = SurrealFSAgent(
    endpoint="http://localhost:8000",
    namespace="agent",
    database="workspace"
)

# File operations
agent.write("/projects/readme.md", "# My Project\nAgent-generated content.")
content = agent.read("/projects/readme.md")
files = agent.list("/projects/")
agent.move("/projects/readme.md", "/archive/readme.md")

# Search across files
results = agent.search("authentication patterns")

# Metadata queries
recent = agent.query("SELECT * FROM file WHERE modified > time::now() - 1h")
```

## Key Features

- POSIX-like file operations (read, write, list, move, delete)
- Persistent storage across agent sessions via SurrealDB
- Full-text and semantic search over file contents
- File metadata as queryable SurrealDB records
- Directory-level permissions and access control
- Designed for multi-agent collaboration on shared workspaces

## Full Documentation

See the main skill's rule file for complete guidance:
- **[rules/surrealfs.md](../../rules/surrealfs.md)** -- architecture, Rust core API, Python agent setup, SurrealDB schema, multi-agent patterns, and deployment
