# Changelog

All notable changes to this project will be documented in this file.
Format based on [Keep a Changelog](https://keepachangelog.com/).

## [1.0.2] - 2026-02-19

### Added
- JavaScript/TypeScript SDK v2.0.0-beta.1 coverage in rules/sdks.md
  - Engine-based architecture (createRemoteEngines, createNodeEngines, createWasmEngines, createWasmWorkerEngines)
  - Multi-session support (newSession, forkSession, await using)
  - Query builder pattern (.fields, .where, .fetch, .content, .merge, .replace, .patch)
  - Query method overhaul (.collect, .json, .responses, .stream)
  - Expressions API (eq, or, and, between, inside, raw, surql template tag)
  - Redesigned live queries (.subscribe, for await, .liveOf)
  - Auto token refresh (renewAccess)
  - User-defined API invocation (.api)
  - Diagnostics API (applyDiagnostics)
  - Codec visitor API (valueDecodeVisitor, valueEncodeVisitor)
  - v1 to v2 migration guide table
- Tracked surrealdb.js v2.0.0-beta.1 (SHA 6383698daccf) in SOURCES.json

## [1.0.1] - 2026-02-19

### Added
- SOURCES.json with commit SHAs, release tags, and dates for all 7 upstream repos
- check_upstream.py script to diff current upstream state against skill snapshot
- Source provenance tables in AGENTS.md, SKILL.md, and README.md with dates
- Detailed Claude Code plugin installation instructions (4 methods)

### Fixed
- KNN operator syntax in AGENTS.md (`<|K,EF|>` takes two numeric params, not distance metric)
- Added `--check` alias for `--quick` flag in doctor.py
- Added exit code 1 on unhealthy status in doctor.py

## [1.0.0] - 2026-02-19

### Added
- Initial release of SurrealDB 3 skill for AI coding agents
- Comprehensive SurrealQL reference (rules/surrealql.md)
- Multi-model data modeling guide (rules/data-modeling.md)
- Graph query patterns (rules/graph-queries.md)
- Vector search and RAG patterns (rules/vector-search.md)
- Security and access control guide (rules/security.md)
- Performance optimization guide (rules/performance.md)
- SDK integration patterns for JS, Python, Go, Rust, Java, .NET (rules/sdks.md)
- Deployment and operations guide (rules/deployment.md)
- Surrealism WASM extension development (rules/surrealism.md)
- Surreal-Sync data migration guide (rules/surreal-sync.md)
- Surrealist IDE guide (rules/surrealist.md)
- SurrealFS AI agent filesystem guide (rules/surrealfs.md)
- Python onboard script with setup wizard and agent capabilities manifest
- Python doctor script for environment health checks
- Python schema script for database introspection and export
- Sub-skills: surrealism, surreal-sync, surrealfs
- CI/CD workflows for validation and release
- Universal compatibility with 30+ AI coding agents
