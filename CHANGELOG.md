# Changelog

All notable changes to this project will be documented in this file.
Format based on [Keep a Changelog](https://keepachangelog.com/).

## [1.4.0] - 2026-05-03

### Major (ecosystem expansion)
- New rule **`rules/surrealmcp.md`** + sub-skill **`skills/surrealmcp/SKILL.md`** covering the official Model Context Protocol server for SurrealDB. Tool catalog (`query`, `select`, `create`, `update`, `merge`, `delete`, `relate`, `live`, `kill`, `schema.introspect`, `schema.tables`, `schema.table`, `info.db`, `info.ns`, `use`, `signin`), stdio + Streamable HTTP transports, host-config snippets for Claude Code, Claude Desktop, Cursor, Codex CLI, OpenCode, Amp, Continue, Windsurf.
- New rule **`rules/editor-tooling.md`** covering `surrealql-language-server`, `surrealql-tree-sitter`, and the official editor extensions: VS Code / Cursor / Windsurf / VSCodium (`surrealql-vsx` grammar, Marketplace + OpenVSX), JetBrains (`surrealql-jetbrains`), Neovim (`surrealql-neovim` + `nvim-treesitter`), Helix (`surrealql-helix`), Sublime Text (LSP-Sublime), Zed (`surrealql-zed`), Emacs (`surrealql-emacs`). Includes `surrealql.toml` config schema and CI lint pattern.
- New rule **`rules/langchain.md`** covering `langchain-surrealdb` (Python) and `@langchain/surrealdb` (JS): vector store, retrievers (similarity / MMR / score-threshold), hybrid retriever (vector + keyword + graph), chat message history, async API, multi-tenant permissioning via DEFINE ACCESS.
- New rule **`rules/surrealml.md`** covering SurrealML model authoring (PyTorch / ONNX / scikit-learn / TensorFlow / HuggingFace), `.surml` artifacts, DEFINE MODEL, `ml::name<version>(...)` invocation, computed-field embeddings, BEFORE-write events, version rollouts via SurrealKit, comparison with Surrealism extensions.
- `rules/sdks.md` expanded with full **Swift**, **Kotlin / JVM**, and **Ruby** SDK sections (installation, connection, auth, CRUD, live queries, framework integration patterns: SwiftUI, Android lifecycle, Rails / ActiveRecord, Sidekiq pooling). Decision matrix updated from 7 columns to 10.
- `rules/deployment.md` adds the **`setup-surreal`** opinionated bootstrap CLI: project scaffolding, storage-engine validation, TLS modes (`none` / `self-signed` / `letsencrypt` / `custom`), Helm values export, scoped-user provisioning, integration map with this skill's scripts, production checklist.

### Major (upstream sync)
- Upstream sync to 2026-05-03 covering five changed repos since the 1.3.1 snapshot:
  - `surrealdb/surrealdb`: +38 commits on `main` past v3.0.5 toward v3.1.0-alpha (HEAD `a97d3af`, 2026-04-29). v3.0.5 remains the latest tagged release.
  - `surrealdb/surrealist`: surrealist-v3.7.4 -> surrealist-v3.8.5 (HEAD `3699b2d`, 2026-05-01). Continued query/explorer/designer iteration; signed release artifacts retained.
  - `surrealdb/surrealdb.py`: v2.0.0-alpha.1 -> v2.0.0 GA (HEAD `6e45a82`, 2026-05-02). SurrealDB 3.x feature support, Python 3.9 dropped, structured error handling, musl Linux wheel support, WS session transaction-id fix, Pydantic Logfire instrumentation example.
  - `surrealdb/surrealdb.go`: +8 commits on `main` since v1.4.0 (HEAD `aef39d3`, 2026-04-30). v1.4.0 still the latest tagged release; pin to v1.4.0 for stability.
  - `surrealdb/surrealkit`: v0.5.0 -> v0.6.0 (HEAD `28f5a1c`, 2026-05-03, pre-release). Iterative patch releases plus procedural-macro publish workflow in CI; CLI surface unchanged.

### Added
- `skills/surrealmcp/SKILL.md` sub-skill manifest mirroring the surrealkit / surrealfs / surreal-sync / surrealism pattern.
- `SOURCES.json` now tracks `surrealdb/surrealmcp`, `surrealdb/surrealml`, `surrealdb/surrealql-language-server`, `surrealdb/surrealql-tree-sitter`, `surrealdb/surrealql-vsx`, `surrealdb/surrealql-jetbrains`, `surrealdb/surrealql-neovim`, `surrealdb/surrealql-zed`, `surrealdb/surrealql-helix`, `surrealdb/surrealql-emacs`, `surrealdb/langchain-surrealdb`, `surrealdb/setup-surreal`, `surrealdb/surrealdb.swift`, `surrealdb/surrealdb.kotlin`, `surrealdb/surrealdb.rb`. All 23 tracked repos resolve via `gh api` -- no 404s in `check_upstream.py`.
- `scripts/onboard.py` capability list and rule index extended with `surrealml`, `surrealmcp`, `editor-tooling`, `langchain`. Decision-tree manifest gains `ml_inference`, `agent_integration`, `editor_setup`, `rag_pipeline` entries.
- `AGENTS.md` decision trees for: deploying ML models, AI agent integration via MCP, editor / IDE setup, LangChain RAG pipelines.

### Changed
- `SOURCES.json`, `SKILL.md`, sub-skill manifests under `skills/*/SKILL.md`, `README.md`, and `AGENTS.md` synced to the 2026-05-03 provenance.
- `rules/sdks.md`: Python SDK section promoted to v2.0.0 GA. Go SDK section calls out the unreleased main HEAD past v1.4.0 explicitly. New Swift / Kotlin / Ruby sections; decision matrix expanded.
- `rules/surrealist.md`: pinned to v3.8.5 with current snapshot date.
- `rules/surrealkit.md` and `skills/surrealkit/SKILL.md`: pinned to v0.6.0 pre-release with continuity note that the public CLI surface is unchanged.
- `rules/deployment.md`: added `setup-surreal` section between configuration flags and Docker deployment.
- `README.md`: feature bullet count updated to 12+ SDKs, new sub-skill sections (SurrealMCP, SurrealML, Editor Tooling, LangChain), architecture tree reflects new `rules/` and `skills/` files.

### Security
- **No regression to declared security posture.** All v1.4.0 changes are documentation-only -- no new scripts, no new binaries vendored, no new third-party network endpoints called by the skill itself, no new credential surface, no new file-write paths, no new shell-execution surface, no obfuscated code, no binary blobs, no `curl | sh` instructions, no minified scripts. The new rule files document upstream tools whose installation continues to use auditable channels (Cargo, Homebrew, npm registry, Docker Hub). CI and Release workflows retain `permissions: contents: read` (Release also explicitly scopes its publish step). `check_upstream.py` continues to use the GitHub API via the `gh` CLI only.
- New rules' security guidance: `rules/surrealmcp.md` recommends scoped DB users (DEFINE USER ... ROLE EDITOR/VIEWER), TLS for HTTP transport, bearer tokens, and never running MCP as root in production. `rules/langchain.md` recommends row-level DEFINE PERMISSIONS for multi-tenant vector stores. `rules/surrealml.md` recommends DEFINE PERMISSIONS on model functions to scope inference. `rules/deployment.md` `setup-surreal` checklist requires non-`memory` storage, non-`none` TLS, and scoped DB users for production.
- All upstream version bumps in this release are equal-or-better on the security axis: surrealdb.go retains the v1.4.0 SQL-injection sanitization in restore (#375); surrealdb.py v2.0.0 GA tightens error-handling types; surrealist v3.8.5 keeps signed release artifacts.
- SKILL.md security frontmatter (`no_network=false note`, `no_credentials=false note`, `no_env_write=true`, `no_file_write=false note`, `no_shell_exec=false note`, `scripts_auditable=true`, `scripts_use_pep723=true`, `no_obfuscated_code=true`, `no_binary_blobs=true`, `no_minified_scripts=true`, `no_curl_pipe_sh=true`) verified accurate after this revision.

## [1.3.1] - 2026-04-10

### Fixed
- ClawHub registry metadata now declares the skill's required binaries and `SURREAL_*` environment variables under `metadata.openclaw`, matching the documented publish contract and eliminating the `metadata: null` registry state from `1.3.0`
- Root `SKILL.md` now carries an explicit top-level `version` field in addition to the repo-local metadata block for better registry compatibility
- Release workflow now publishes through the supported `clawhub` CLI flow instead of the dead `api.clawhub.ai/v1/skills/publish` endpoint

### Changed
- Version metadata bumped to `1.3.1` across the root manifest, sub-skills, AGENTS.md, README badge, and SOURCES.json

## [1.3.0] - 2026-04-10

### Major
- SurrealDB v3.0.5: documented `REMOVE CONFIG`, wider `ALTER` coverage, planner pushdown fixes, `$parent` fixes, computed field fixes, edge query ordering fixes, GraphQL literal fields, and related patch work
- SurrealKit v0.5.0 added as a first-class part of the skill: desired-state schema sync, rollout-based migrations, seeding, and declarative database/API tests
- Release and CI workflows hardened: explicit least-privilege permissions, version-consistency validation, smoke tests, and no in-workflow manifest mutation before publish

### Fixed
- `scripts/check_upstream.py`: short baseline SHAs now compare correctly against full GitHub commit SHAs
- `scripts/check_upstream.py`: falls back to latest Git tag when a repo does not publish GitHub Releases
- `scripts/doctor.py` and `scripts/schema.py`: normalize user-facing HTTP endpoints to SurrealDB WebSocket RPC URLs before connecting
- `scripts/schema.py`: restored the documented `introspect`, `tables`, and `table` commands
- `scripts/onboard.py`: version now comes from root `SKILL.md`, and the agent manifest now reflects live prerequisites and the full script/rule set

### Changed
- README, AGENTS.md, SKILL.md, and SOURCES.json synced to upstream state as of 2026-04-10
- Added `rules/surrealkit.md` and `skills/surrealkit/SKILL.md`
- Surrealist docs updated to v3.7.4; JavaScript SDK docs updated to v2.0.3
- Security metadata corrected: file writes are declared accurately, remote shell installer references removed from active documentation, and publish workflow now validates repository content instead of editing it at release time

## [1.2.1] - 2026-03-13

### Major
- SurrealDB v3.0.4: 20 fixes/features including GraphQL Subscriptions (#7027),
  BM25 search::score() compaction fix (#7057), HNSW index compaction fix (#7077),
  UPSERT conditional count fix (#7056), LIMIT with incomplete WHERE fix (#7063),
  v2 subcommand for migration assistance (#7058), concurrent startup retry (#7055),
  distributed task lease race fix (#6501), and performance improvements (#7018)
- JS SDK v2.0.2: streamed imports/exports (#563), blob import support (#568),
  single value for StringRecordId (#569)
- Surrealist v3.7.3: PrivateLink support, streamed import/export, org ID in
  settings, node rendering perf, dataset rename, improved ticket display
- Surreal-Sync: SurrealDB v3 compatibility, PostgreSQL foreign key relations,
  TOML config, Neo4j relationship fix, improved test infrastructure

### Changed
- SOURCES.json synced to HEAD 2026-03-13 (all 7 repos updated)
- rules/surrealql.md: v3.0.4 patch notes section (20 items)
- rules/sdks.md: JS SDK v2.0.2 changes
- rules/surrealist.md: v3.7.3 version and features
- rules/surreal-sync.md: v3 compatibility notes

## [1.2.0] - 2026-03-03

### Major
- SurrealDB v3.0.2 patch release (2026-03-03): 13 fixes/features documented in
  rules/surrealql.md including None-on-missing-record, bind parameter resolution
  in MATCHES operator, datetime setter functions, configurable CORS, --tables-exclude
  export flag, compound unique index fix, DELETE live event permissions, DEFINE
  FUNCTION parsing fix, transaction timeout enforcement, executor optimizations
- Go SDK v1.4.0: SurrealDB v3 structured error handling (new ServerError type),
  identifier sanitization in restore to prevent SQL injection
- JS SDK: RPC query stat duration parsing fix (#560)

### Changed
- SOURCES.json synced to HEAD 2026-03-03 (surrealdb d454269ecb11, surrealdb.js
  501b167b2155, surrealdb.go a7bf54bc9487)
- Docker image tags use v3 (auto-tracks v3.0.2)
- v3.1.0-alpha tracking updated (error chaining, SurrealValue, timestamp refactor)

## [1.1.1] - 2026-02-26

### Fixed
- Python SDK release corrected from v2.0.0 to v2.0.0-alpha.1 (pre-release alpha,
  not GA). Python 3.9 dropped; minimum is now 3.10. Added Logfire instrumentation note.

### Added
- SurrealDB v3.1.0-alpha behavior change: SELECT on non-existent records now returns
  NONE instead of error (#6978). Documented in rules/surrealql.md with migration note.

### Changed
- SOURCES.json synced to HEAD 2026-02-26 (surrealdb fa22ecf0ae93, surrealdb.py b21302c05565)
- AGENTS.md: added context comment on production 0.0.0.0 bind address

## [1.1.0] - 2026-02-25

### Major
- JavaScript SDK v2.0.0 GA released (no longer beta). Updated from beta tag to
  stable: `npm install surrealdb` (not @beta). Full SurrealDB 3.0.1 support,
  client-side transactions, multi-session, query builder, streaming responses.
- Python SDK v2.0.0 released. WebSocket session transaction ID fix, musl Linux
  support for Alpine/containers, improved error handling, README cleanup.

### Changed
- rules/sdks.md: JS v2 section title changed from "beta" to "GA -- recommended
  for new projects". Install commands changed from surrealdb@beta to surrealdb.
  All @surrealdb/wasm@beta and @surrealdb/node@beta tags removed.
- rules/sdks.md: Python SDK updated to v2.0.0 with changelog
- rules/surrealql.md: v3.1.0-alpha tracking updated with error chaining
  infrastructure (#6969), SurrealValue derive convenience (#6970), wasmtime
  update (#6973)
- SOURCES.json: All repos synced to HEAD 2026-02-25. Removed surrealdb.js@beta
  entry (v2 is now GA). surrealdb.js release v2.0.0, surrealdb.py release v2.0.0.
- Additional credential warning markers on remaining unwarned root/root examples
  in SKILL.md workflow section and AGENTS.md decision tree
- deployment.md: --bind flag default annotated with local dev recommendation

## [1.0.6] - 2026-02-24

### Added
- SurrealDB v3.0.1 patch notes in rules/surrealql.md: duration arithmetic, computed
  field index prevention, record ID dereference fix, error serialization, GraphQL
  string enum fix, root user permission fix, parallel index compaction, WASM compat,
  RouterFactory trait for embedders
- v3.1.0-alpha tracking notes (main branch: planner tidy-up, test fixtures, code coverage)
- JS SDK v2.0.0-beta.2 changes: ne (!=) operator, error cause property, createWorker
  factory for Vite-compatible Web Worker engines, minimum SurrealDB version bump to 2.1.0
- Python SDK error handling improvements (#233)

### Changed
- All upstream repos synced to HEAD as of 2026-02-24
- SOURCES.json: surrealdb release updated v3.0.0 -> v3.0.1, added main_tracking field
- SOURCES.json: surrealdb.js@beta release updated beta.1 -> beta.2
- Docker image tags updated from v3.0.0 to v3 (tracks latest v3.x)
- AGENTS.md: fixed remaining 0.0.0.0 bind address to 127.0.0.1
- rules/deployment.md: fixed remaining 0.0.0.0 bind to 127.0.0.1 with comment
- rules/sdks.md: createWasmWorkerEngines example updated for beta.2 createWorker factory
- rules/sdks.md: added ne operator to Expressions API imports

## [1.0.5] - 2026-02-24

### Added
- Native GitHub Copilot agent skill support (.github/skills/surrealdb/SKILL.md)
  - Follows the open Agent Skills standard (agentskills.io)
  - Auto-loads in VS Code, Copilot CLI, and Copilot coding agent when SurrealDB context detected
  - Available as `/surrealdb` slash command in Copilot chat
  - Progressive disclosure: metadata -> instructions -> rule files on demand
  - Supports project-level (.github/skills/) and personal (~/.copilot/skills/) installation
  - Includes `argument-hint` for guided slash command usage
  - References all 12 rule files via relative paths for Copilot resource loading
  - Quick reference section with SurrealQL essentials for immediate context

### Changed
- README: replaced "append AGENTS.md to copilot-instructions.md" with native Copilot
  agent skills instructions (3 install methods: project, personal, /skills menu)
- README: added Cursor .cursor/skills/ integration (same Agent Skills standard)
- Upstream sync to 2026-02-24:
  - surrealdb/surrealdb: +2 commits (error serialization fix, CI fix)
  - surrealdb/surrealist: +1 commit (strict sandbox option fix)
  - surrealdb/surrealdb.js: +2 commits (version bumps)
- SOURCES.json baselines updated to current HEAD SHAs

## [1.0.4] - 2026-02-22

### Security Fixes (addressing OpenClaw/VirusTotal scan findings)
- SKILL.md frontmatter: changed no_network and no_credentials to false with
  explanatory notes (scripts DO connect to user-specified endpoints)
- SKILL.md frontmatter: added requires.binaries declaring surreal, python3, uv, docker
- SKILL.md frontmatter: added requires.env_vars declaring all SURREAL_* vars
  with sensitive: true on SURREAL_USER and SURREAL_PASS
- Replaced remote shell installer instructions with brew/package manager alternatives
  in SKILL.md, AGENTS.md, README.md, and rules/deployment.md
- Added security notes on remote shell installers (download-and-review alternative documented)
- Added credential warnings on all root/root examples across all files
- Changed bind address from 0.0.0.0 to 127.0.0.1 in quick start examples
- Added SurrealQL injection prevention: _sanitize_identifier() in schema.py
  validates table names against [a-zA-Z_][a-zA-Z0-9_]* before query interpolation
- surrealfs sub-skill: added Security Considerations section covering telemetry
  opt-out (LOGFIRE_SEND_TO_LOGFIRE=false), HTTP binding, pipe command risks,
  sandboxing, credential scoping
- surrealfs sub-skill: added requires.env_vars and security block to frontmatter
- README: corrected security properties table (no_network=false, no_credentials=false)
- README: added Required Environment Variables table with sensitivity markers
- README: added Required Binaries table
- README: added Script Safety section

## [1.0.3] - 2026-02-22

### Added
- Nightly upstream freshness check GHA workflow (.github/workflows/upstream-check.yml)
  - Runs at 06:00 UTC daily, auto-creates/updates GitHub issue when repos drift
  - Manual trigger via workflow_dispatch
- ClawHub/OpenClaw publishing (clawhub.ai registry)
- Security metadata in SKILL.md frontmatter (no_network, no_credentials, scripts_auditable, etc.)
- Registries section in README with skills.sh, ClawHub, OpenClaw install commands
- Security properties table in README
- GitHub topics: openclaw, clawhub, agentskills (replacing lower-value topics)
- Opened surrealdb/surrealdb#6958 for community resource listing

### Changed
- Synced upstream sources to latest HEAD (snapshot 2026-02-22):
  - Surrealist v3.7.1 -> v3.7.2 (migration export fix, misc UI fixes)
  - surrealdb.js WASM SDK updated to 3.x, WebWorker Vite compatibility fix
- Updated provenance tables in AGENTS.md, SKILL.md, README.md
- Updated sub-skills with provenance metadata and corrected upstream CLI syntax
- Updated repo description and homepage on GitHub

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
