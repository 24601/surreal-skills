---
name: surreal-sync
description: "Data migration and synchronization to SurrealDB from MongoDB, PostgreSQL, MySQL, Neo4j, Kafka, and JSONL. Full and incremental CDC sync. Part of the surreal-skills collection."
license: MIT
metadata:
  version: "1.0.0"
  author: "24601"
  parent_skill: "surrealdb"
---

# Surreal-Sync -- Data Migration and Synchronization

Surreal-Sync provides patterns and tooling for migrating data into SurrealDB from a variety of source systems. It supports both full initial loads and incremental change data capture (CDC) synchronization.

## Supported Sources

| Source | Full Sync | Incremental CDC | Notes |
|--------|-----------|----------------|-------|
| MongoDB | Yes | Yes | Via change streams |
| PostgreSQL | Yes | Yes | Via logical replication / WAL |
| MySQL | Yes | Yes | Via binlog |
| Neo4j | Yes | No | Graph-to-graph mapping |
| Apache Kafka | Yes | Yes | Topic consumption |
| JSONL Files | Yes | N/A | Batch import from newline-delimited JSON |

## Quick Start

```bash
# Full sync from PostgreSQL
surreal-sync --source postgres --conn "postgresql://user:pass@localhost/mydb" \
  --target http://localhost:8000 --ns prod --db main --mode full

# Incremental CDC from MongoDB
surreal-sync --source mongo --conn "mongodb://localhost:27017/mydb" \
  --target http://localhost:8000 --ns prod --db main --mode cdc

# Batch import from JSONL
surreal-sync --source jsonl --file data.jsonl \
  --target http://localhost:8000 --ns prod --db main --table imports
```

## Key Features

- Automatic schema inference and SurrealDB table creation
- Record ID mapping from source primary keys
- Relationship extraction and graph edge creation
- Configurable batch sizes and parallelism
- Resumable sync with checkpoint tracking

## Full Documentation

See the main skill's rule file for complete guidance:
- **[rules/surreal-sync.md](../../rules/surreal-sync.md)** -- source configuration, schema mapping, CDC setup, conflict resolution, and production deployment
