# Changelog

All notable changes to this project will be documented in this file.
Format based on [Keep a Changelog](https://keepachangelog.com/).

## [1.5.2] - 2026-05-05

### Fixed (atomic-protocol patch — v1.5.1 Pi-only re-audit CRIT remediation)

A second Pi+DeepSeek-V4-Pro:xhigh adversarial pass over the same five
rules patched in v1.5.1 (`rules/data-modeling.md`, `rules/security.md`,
`rules/vector-search.md`, `rules/performance.md`,
`rules/graph-queries.md`) returned **2 GO + 2 CONDITIONAL GO + 1 NO-GO**
with **7 CRITs** total — every one introduced or missed by the v1.5.1
fix surgery. v1.5.2 patches every CRIT. The empirical fix-patch-drift
pattern noted in `~/CLAUDE.md` ("each rev closes prior CRITs but
introduces 1-2 new ones from fix surgery") played out as expected,
mostly concentrated in the file with the largest pass-1 patch surface
(`rules/performance.md`, 8 sites).

- **`rules/performance.md`** (3 CRITs — all v1.5.1-introduced) — The
  v1.5.1 EXPLAIN-output rewrite invented two operator names that do
  not appear in the v3.0.5 operator catalog (`RangeScan`, standalone
  `Iterate`) and asserted that the clause-form `… EXPLAIN` does not
  produce `operation: 'Iterate Table'` / `operation: 'Iterate Index'`
  — but it does. The actual upstream surface is **two distinct
  output shapes** depending on syntax form: the clause form
  (`SELECT … EXPLAIN`) emits `operation:` rows with values like
  `'Iterate Table'` / `'Iterate Index'` / `'Fetch'`, and the statement
  form (`EXPLAIN SELECT …`) emits `operator:` rows from the planner
  scan catalog (`Scan`, `TableScan`, `IndexScan`, `CountScan`,
  `IndexCountScan`, `FullTextScan`, `GraphEdgeScan`, `ReferenceScan`,
  `KnnScan`, `UnionIndexScan`). The Query Performance section now
  documents both shapes explicitly, removes the hallucinated
  `RangeScan` and standalone `Iterate` rows, and updates the §"Query
  Timing" examples and §"Common Bottlenecks" table cell to label the
  output as clause-form `operation:` (or to show both forms) so the
  rest of the file no longer contradicts the operator-name section.

- **`rules/graph-queries.md`** (1 CRIT) — §"Setting Properties on
  Edges" included a third example using `RELATE … MERGE { … }`. v3
  `RELATE` grammar is `RELATE [ ONLY ] @from -> @table -> @to
  [ CONTENT @value | SET @field = @value … ] [ RETURN … ]
  [ TIMEOUT @duration ]` — there is no `MERGE` clause on `RELATE`.
  The example is replaced with the correct way to update an existing
  edge's properties (`UPDATE knows SET last_interaction = time::now()
  WHERE in = person:alice AND out = person:bob`) and a precision
  comment quotes the verified RELATE grammar.

- **Cross-fix in `rules/surrealql.md`** (3 CRITs) — The vector-search
  re-audit surfaced three function-path hallucinations in the
  language reference that v1.4.4 missed:
  `vector::distance::cosine`, `vector::distance::jaccard`,
  `vector::distance::pearson`. v3.0.5 registers these only under
  `vector::similarity::*`. The three offending lines are removed and
  a precision note now points readers at the similarity functions
  (and explains how to derive a distance-shaped value:
  `1 - vector::similarity::cosine(...)` etc.). The valid distance
  functions (`chebyshev` / `euclidean` / `hamming` / `manhattan` /
  `minkowski`) remain in place.

`rules/data-modeling.md` and `rules/security.md` returned no CRITs in
re-audit. `rules/vector-search.md` itself was clean (its pass-1 LM/M0
fix held); the only `vector::*` issue lived in `surrealql.md` and is
patched there. Pass-2 IMPORTANTs (e.g. data-modeling's UUID-version
specificity, security's `WITH REFRESH` / RECORD-WITH-JWT-URL gaps,
performance's `REBUILD INDEX` / `CONCURRENTLY` / `DEFER`
under-documentation) are deferred — they are documentation polish or
gaps, not contradictions of upstream — and tracked in the residual
risk lists of the per-rule re-audit reports at
`/tmp/pi-{rule}-v1.5.1-out.md`.

### Migration

Consumers who copied either of the v1.5.1-introduced performance
operator names (`RangeScan` or standalone `Iterate`) need to substitute
the real upstream operator names per the new dual-format block in
`rules/performance.md`. Consumers who copied the `RELATE … MERGE { … }`
form from `rules/graph-queries.md` need to switch to `UPDATE` on the
edge record. Consumers calling `vector::distance::cosine`,
`vector::distance::jaccard`, or `vector::distance::pearson` from
`rules/surrealql.md` need to migrate to the corresponding
`vector::similarity::*` functions (subtract from 1 if a distance-shaped
value is required). The machine-checked version-consistency CI gate
continues to apply.

## [1.5.1] - 2026-05-05

### Fixed (atomic-protocol patch — v1.5.0 Pi-only adversarial-audit CRIT remediation)

A Pi+DeepSeek-V4-Pro:xhigh adversarial audit run against
`rules/data-modeling.md`, `rules/security.md`, `rules/vector-search.md`,
`rules/performance.md`, and `rules/graph-queries.md` after v1.5.0
returned 5/5 NO-GO with 21 CRITs total — hallucinations beyond the
mechanically-grepped v1.4.4 patterns. v1.5.1 patches every CRIT.

- **`rules/vector-search.md`** (2 CRITs) — HNSW parameter table swapped
  `LM` and `M0`. `LM` was documented as "Max connections at layer 0,
  default `2*M`"; the parser actually treats `LM` as the **Minkowski
  distance order** (only meaningful with `DIST MINKOWSKI`) and `M0`
  as the layer-0-connections clause (default `2*M`). The full HNSW
  parameter list now lists both `M0` and `LM` with their correct
  semantics, and a precision note flags the pre-v1.5.1 conflation so
  copied snippets can be repaired.
- **`rules/data-modeling.md`** (2 CRITs) — (a) the §"Schema Modes"
  table conflated schema enforcement (`SCHEMAFULL` / `SCHEMALESS`)
  with table type markers (`TYPE NORMAL` / `TYPE RELATION` /
  `TYPE ANY`) into a single 5-row "modes" table, implying mutual
  exclusivity. They are orthogonal — a table can be `TYPE RELATION
  SCHEMAFULL`. The section now splits into two tables with a note
  on combination. (b) The social-feed example used
  `->follows->user->wrote->post.*` against a schema that defines
  only `follows` and `likes` edges — no `wrote` edge — so the query
  failed at runtime. The example now uses the record-link form
  (`SELECT * FROM post WHERE author IN (SELECT VALUE ->follows->user
  FROM user:alice)`) and adds a comment on what to define if you
  want the `wrote`-edge form instead.
- **`rules/security.md`** (3 CRITs) — (a) §"API Key Authentication"
  documented `DEFINE ACCESS api_access ON DATABASE TYPE API KEY` —
  there is no `TYPE API KEY` in v3.0.5; the section is renamed
  "Bearer-Token Authentication" and now documents the actual
  mechanism: `DEFINE ACCESS … TYPE BEARER FOR [USER | RECORD]` plus
  `ACCESS <name> GRANT FOR USER|RECORD <subject>`. (b) `TYPE JWT`
  examples used `DURATION FOR TOKEN`, which is invalid on JWT
  access — JWT tokens are issued externally, so SurrealDB only
  accepts `DURATION FOR SESSION` here. All three JWT examples were
  rewritten to `DURATION FOR SESSION 12h`. (c) `WITH ISSUER KEY` was
  missing from the JWT-with-record-binding examples — this is
  required for SurrealDB to *issue* tokens under that access method
  rather than only verify them. Both relevant examples now include
  the `WITH ISSUER KEY` clause with prose on its purpose.
- **Cross-fix in `rules/surrealql.md`** — the same `TYPE JWT` +
  `DURATION FOR TOKEN` invalid combination appeared in the DEFINE
  ACCESS examples (lines ~507–514). Both were rewritten to
  `DURATION FOR SESSION` to keep the language reference and the
  security rule in sync.
- **`rules/performance.md`** (8 CRITs) — (1) EXPLAIN output
  documented `Iterate Table` / `Iterate Index` operator names; the
  actual user-facing operator names are `TableScan`, `IndexScan`,
  `RangeScan`, `Iterate`. The interpretation block now lists the real
  names. (2) The `WITH` clause for index hints (`WITH NOINDEX`,
  `WITH INDEX <name>`) was undocumented — added a "Index Hints" sub-
  section under Query Optimization. (3) The `surrealkv://` start
  example included `surreal start file:///var/data/surreal.db` as
  a synonym; `file://` is deprecated in v3 and emits a deprecation
  warning. The `file://` line is removed and the prose flags it
  explicitly. (4) `surreal start --rocksdb-cache-size 4GB` does not
  exist; the cache section now points at the env-var surface
  (`SURREAL_ROCKSDB_BLOCK_SIZE` etc.) and lists the verified
  `surreal start` flags. (5) `surreal start --max-connections 1000`
  also does not exist; the connection-limits section now describes
  bounding concurrency at the proxy / OS layer instead. (6) "SurrealKV
  (default in SurrealDB 3.x for file-based storage)" implied
  automatic substitution; rephrased to "recommended for file-based
  storage" with a note that the no-arg default is `memory`. (7)
  §"Parallel Query Execution" conflated the `SELECT … PARALLEL`
  clause (intra-query worker parallelism) with multi-statement
  request batching (round-trip reduction). Split into two distinct
  subsections. (8) The `FETCH` clause for record-link resolution was
  not discussed at all — added a "FETCH vs Subquery" subsection.
- **`rules/graph-queries.md`** (6 CRITs) — (1) §"Shortest Path
  Queries" claimed *"SurrealDB does not have a native shortest-path
  function"* and built a hand-rolled BFS. v3 has a native
  `..+shortest=target` modifier (with optional `+path`) — the entire
  hand-rolled BFS is replaced with the native form. (2) The §"Recursive
  Traversal Patterns" section showed only fixed-hop chaining and
  missed the v3 mandatory destructuring depth/range syntax
  (`person:alice.{..3}->reports_to->person`,
  `person:alice.{1..3}->reports_to->person`,
  `org:company.{..}.children`). The section now leads with the
  destructuring form and keeps fixed-hop chains as a fallback.
  (3) `.@` recursive destructuring (which builds nested trees in a
  single expression) was missing entirely; added a sub-section with
  examples for both edge traversals and `REFERENCE` link fields.
  (4) The §"Aliased Traversal" example used `AS` *inside* a
  parenthesised arrow filter
  (`->(knows WHERE since > d'2023-01-01' AS recent_connections)->person`),
  which no official v3 test exercises. The example is rewritten to
  the SELECT-projection-position form and a note flags the previous
  form as unverified. (5) Wildcard edge traversal (`->?`, `<-?`,
  `<->?`, `->?->?`) was undocumented; added a sub-section. (6) Path
  modifiers `+collect`, `+path`, `+inclusive` (which compose with
  the depth/range modifier to change what the traversal returns)
  were undocumented; added a sub-section with examples.

The audit was run against v1.4.5 HEAD (`f83ca4e`); each rule's verdict
came from a separate Pi process to avoid cross-rule pollution. Pi
output files are at `/tmp/pi-{rule}-audit.md` for traceability — these
were not committed but are referenced from the v1.5.1 patch surgery.

### Migration

No consumer code changes for callers using the language reference
(`rules/surrealql.md`) — the v1.5.1 cross-fix there only narrows
already-broken examples. Consumers who copied any of the bullets
above (especially HNSW snippets using `LM` for layer-0 connections,
DEFINE ACCESS using `TYPE API KEY`, JWT access using `DURATION FOR
TOKEN`, `surreal start` invocations using `--rocksdb-cache-size` /
`--max-connections` / `file://`, or hand-rolled BFS for shortest
paths) need to apply the corrections noted in each bullet. The
machine-checked version-consistency CI gate continues to apply.

## [1.5.0] - 2026-05-05

### Added (deferred-verification milestone — close v1.4.1 deferrals)

The v1.4.1 patch shrank `rules/editor-tooling.md`, `rules/surrealmcp.md`,
and `rules/surrealml.md` to verified content only and explicitly
deferred per-extension/tool/API detail to v1.5.0. v1.5.0 closes that
deferral by inspecting actual upstream source for each surface and
restoring fully-grounded tables:

- **`rules/editor-tooling.md`** — restored per-editor tables from
  pinned upstream source: `surrealdb/surrealql-language-server@v0.1.2`
  (full `surrealql.*` workspace settings, env-var fallbacks, server
  capabilities, build instructions), `surrealdb/surrealql-vsx@v0.3.0`
  (`surrealdb.surrealql` VS Code extension is grammar+snippets only —
  no commands, no settings, no LSP wiring), `surrealdb/surrealql-zed@v0.1.0`
  (`surrealdb-surrealql` extension config, LSP discovery, asset names),
  and `surrealdb/surrealql-jetbrains` head (plugin id
  `com.surrealdb.surql-jetbrains`, settings page **Settings → Tools →
  SurrealQL**, LSP4IJ wiring). Confirmed `surrealql-language-server` is
  the canonical LSP that first-party extensions wire to (Zed + JetBrains
  both shell out to it by name); `surql-lsp` is a separate community
  crate. Confirmed no first-party Sublime / Neovim / Helix / Emacs
  packages exist — wire-it-yourself sections updated accordingly.
- **`rules/surrealmcp.md`** — restored full tool argument schema table
  from `surrealdb/surrealmcp@v0.4.0`'s `src/tools/mod.rs` `*Params`
  structs. Documents all 20 tools (8 database CRUD, 6 connection
  management, 6 cloud) with required vs optional args, and notes the
  `upsert`/`update` `patch_data`/`merge_data`/`content_data`/`replace_data`
  exclusivity precedence.
- **`rules/surrealml.md`** — restored full Python `SurMlFile`
  constructor + builder API from the published `surrealml 0.0.4` wheel.
  Documents the `Engine` enum (5 variants, with `NATIVE` flagged as
  declared-but-unsupported), the constructor signature, all 8 builder
  methods (`add_column`, `add_normaliser`, `add_output`,
  `add_description`, `add_version`, `add_name`, `add_author`,
  `save`/`to_bytes`), and the static `load`/`upload` + inference
  (`raw_compute`, `buffered_compute`) entry points. Re-asserts what
  does NOT exist: no `from_pytorch`/`from_onnx`/`from_sklearn`/
  `from_keras`/`from_hf` factories, no `ModelMeta`, no `[hf]` extra,
  and no SurrealQL-side `DEFINE MODEL` / `INFO FOR MODEL` / `REMOVE
  MODEL` / `ml::name<version>(...)` invocation form upstream.

### Verified upstream (clones inspected at the v1.5.0 cut)

- `surrealdb/surrealmcp@v0.4.0` (Rust, MCP server)
- `surrealdb/surrealql-language-server@v0.1.2` (Rust, LSP)
- `surrealdb/surrealql-vsx@v0.3.0` (TypeScript, VS Code grammar+snippets)
- `surrealdb/surrealql-zed@v0.1.0` (Rust, Zed extension)
- `surrealdb/surrealql-jetbrains` head (Kotlin, JetBrains plugin)
- `surrealdb/surrealql-tree-sitter` head (tree-sitter grammar)
- `surrealml 0.0.4` (Python wheel from PyPI; `surrealdb/surrealml`
  GitHub repo's tags do not match PyPI release names — wheel was the
  authoritative artefact)
- `surrealdb/langchain-surrealdb@v0.2.1` (Python, cloned but not yet
  used to expand `rules/langchain.md` — that expansion is queued for
  a future release)

### Migration

No consumer code changes. Existing skill consumers using the v1.4.x
shrunken rule files are unaffected; the new content adds detail
without removing or renaming any prior surface. Expanded sections are
strictly additive over the v1.4.5 verified-only baseline.

## [1.4.5] - 2026-05-05

### Fixed (atomic-protocol patch — propagate v1.4.4 SurrealQL corrections to dependent rules)

After v1.4.4 corrected the foundational `rules/surrealql.md`, a
mechanical grep across the rest of the rule set found the same
v1.4.4-class CRIT patterns (`SEARCH ANALYZER` / `MTREE` /
`EXPLAIN FULL` / `string::is::*`) had also propagated into:

- **`rules/data-modeling.md`** -- 16x `SEARCH ANALYZER`, 7x
  `MTREE` (asserted as supported syntax with `DIMENSION` and
  `CAPACITY` parameters), 1x `string::is::email`, plus the
  HNSW/MTREE column in the migration-target table.
- **`rules/security.md`** -- 2x `string::is::email`.
- **`rules/vector-search.md`** -- 1x `SEARCH ANALYZER`. (The
  `MTREE` retraction note already in this file was already
  correct and was retained.)
- **`rules/performance.md`** -- 2x `SEARCH ANALYZER`, 3x
  `EXPLAIN FULL`.

All instances replaced via mechanical pass to match the verified
v3 forms: `FULLTEXT ANALYZER`, HNSW (or `<|K,METRIC|>` brute-force
operator), `EXPLAIN [ ANALYZE ] [ FORMAT TEXT | JSON ] @statement`,
`string::is_*`. The `MTREE Index` section in `data-modeling.md` was
rewritten as an "Exact kNN (no index) -- v3" section pointing at
the brute-force operator.

No 3-way reviewer pass was run for this patch -- the changes were
mechanical replacements of already-verified-wrong patterns from
v1.4.4. The `scripts/check_version_consistency.py` machine-check
catches version-row drift on every CI run from this release
forward.

### Migration
No consumer code changes. The same migration guidance from v1.4.4
applies to anyone who copy-pasted from `rules/data-modeling.md` /
`rules/security.md` / `rules/vector-search.md` /
`rules/performance.md` in v1.4.0 through v1.4.4.

## [1.4.4] - 2026-05-05

### Fixed (atomic-protocol patch — adversarial-review NO-GO findings, batch-4: foundational language reference)

After v1.4.3 shipped, an adversarial review of `rules/surrealql.md`
(the foundational SurrealQL language reference -- highest-impact
failure mode in this skill) returned **NO-GO** with **10 CRITICAL**
findings + 18 IMPORTANT + 9 MINOR. Pi (`deepseek-v4-pro:xhigh`) ran
direct upstream verification against
`surrealdb/docs.surrealdb.com@main/src/content/reference/query-language/...`
on 2026-05-05; Cursor flagged additional internal-consistency drift
between `rules/surrealql.md` and `rules/surrealism.md`; Codex hit
context-window exhaustion on the 2096-line input and produced no
output (false negative; not used).

The same generation-batch failure mode that produced 50+
hallucinations across the rules patched in v1.4.1 / v1.4.2 / v1.4.3
also produced wholesale syntax errors in the foundational reference.
Most of these errors are pre-v3.0.0-beta SurrealQL syntax that the
generation pass inherited from older training data without
verifying against the current grammar.

#### Pervasive syntax corrections
- **`SEARCH ANALYZER` -> `FULLTEXT ANALYZER`** across every `DEFINE INDEX` example, prose mention, and Best Practices section. Upstream `define/indexes.mdx` confirms: "Before SurrealDB version 3.0.0-beta, the `FULLTEXT ANALYZER` clause used the syntax `SEARCH ANALYZER`."
- **`time::from::*` -> `time::from_*`** across every example. Upstream: "Since version 3.0.0-beta, the `::from::` functions now use underscores."
- **`string::is::*` -> `string::is_*`** across every example. Upstream: "Since version 3.0.0-beta, the `::is::` functions now use underscores."
- **`math::PI` / `math::E` / `math::TAU` / `math::INF` / `math::NEG_INF` -> lowercase** (`math::pi`, `math::e`, `math::tau`, `math::inf`, `math::neg_inf`).

#### Fabricated syntax retractions
- **`MTREE` index type**: removed entirely. Upstream `DEFINE INDEX` grammar defines exactly ONE vector index type (`HNSW`); the `MTREE` keyword and its `CAPACITY` parameter were not in the grammar at any v3 version. Replaced with a note pointing at HNSW + the `<|K,METRIC|>` brute-force kNN operator.
- **`string::trim::start` / `string::trim::end`**: removed. Only `string::trim` exists upstream.
- **`math::log2` / `math::log10`**: removed. Upstream has `math::log` (with optional base) and `math::ln`; use `math::log(x, 2)` or `math::log(x, 10)`.
- **`EXPLAIN FULL`**: replaced with the verified standalone form `EXPLAIN [ ANALYZE ] [ FORMAT TEXT | JSON ] @statement`. The `FULL` keyword does not exist in the upstream grammar; the rule's clause-form `SELECT ... EXPLAIN` was retained as the alternate form.
- **`?.` JS-style optional chaining as an operator**: removed from operators table. Replaced with the verified upstream form `spouse.?.name` (period-before-question-mark) on the appropriate access example.
- **`LIKE` / `NOT LIKE` operators**: removed from operators table. Not present in upstream `operators.mdx`, not in the `ifelse` /`where` docs, not in the parser keyword list. Use `~` (fuzzy match) and `CONTAINS` operators.

#### Access-statement structural fix
- **`WITH ISSUER KEY`** moved out of the standalone `TYPE JWT` access example into a `TYPE RECORD WITH JWT` example (the only context in which it is a valid clause per upstream `define/access/record.mdx`). The previous `DEFINE ACCESS api_auth ON NAMESPACE TYPE JWT ... WITH ISSUER KEY ...` would not parse.

#### Type system note
- The `union<...>` and `array<T, N>` type constructors that earlier
  drafts (re-)introduced are not present in upstream; literal types
  use the `|` syntax (`datetime | uuid | "N/A"`). Verified in
  current `Complex Types` table: only `array<T>`, `set<T>`,
  `option<T>`, `record<T>` are documented.

#### Deferred to v1.5.0 (acknowledged gaps; not blocking ship)
- `DEFINE API`, `DEFINE CONFIG`, `DEFINE ACCESS ... TYPE BEARER`
  full sections.
- `ALTER` statement family (16 sub-statements: `ALTER TABLE`,
  `ALTER FIELD`, `ALTER INDEX`, etc.).
- Standalone `ACCESS` statement (`GRANT` / `SHOW` / `REVOKE` /
  `PURGE`) for bearer-grant management.
- `COUNT` index type, `CONCURRENTLY` and `DEFER` clauses on `DEFINE INDEX`.
- `REFERENCE` / `DEFAULT ALWAYS` clauses on `DEFINE FIELD`.
- `INSERT RELATION` variant.
- `DEFINE FUNCTION` `-> @type` return-type syntax + `PERMISSIONS` clause.
- Missing function categories: `encoding::*`, `bytes::*`, `file::*`, `api::*`, `sequence::*`, `set::*`, plus several `string::*`, `time::*`, `math::*`, `search::*` individual functions.

These deferrals are documented in `docs/v1.5.0-roadmap.md` style
(see CHANGELOG history) -- the rule body still teaches the
verified-correct primary surface.

### Security posture
- No new scripts, binaries, or third-party endpoints. All upstream
  verification was via public read-only fetches against
  `docs.surrealdb.com` on 2026-05-05. No new credential surface.
- Removing the wrong access-method `WITH ISSUER KEY` placement
  closes a SurrealQL-failure-at-parse-time surface where a
  developer copy-pasting from v1.4.0 / v1.4.1 / v1.4.2 / v1.4.3
  docs would build code that fails to define the access at all.

### Migration
No consumer code changes. Rule-file content has been replaced;
consumers that copy-pasted from earlier versions should re-pin to
v1.4.4 and re-derive any code from the corrected rule text. In
particular: switch every `DEFINE INDEX ... SEARCH ANALYZER ...` to
`FULLTEXT ANALYZER`; rename every `time::from::X` to
`time::from_X`; rename every `string::is::X` to `string::is_X`;
delete any `MTREE` index definitions and rebuild as `HNSW` (or use
`<|K,METRIC|>` brute-force kNN); remove any `EXPLAIN FULL` /
`?.` / `LIKE` / `NOT LIKE` usage; relocate any `WITH ISSUER KEY`
clause from a JWT-typed access definition into the corresponding
RECORD-typed access.

### Tooling
- `scripts/check_version_consistency.py` (added in commit `b842203`
  before this release) is now wired into `ci.yml` so future
  version-drift across `SKILL.md` / sub-skills / `SOURCES.json` /
  `README.md` badge / `CHANGELOG.md` / `AGENTS.md` is caught
  mechanically on every PR.

## [1.4.3] - 2026-05-05

### Fixed (atomic-protocol patch — adversarial-review NO-GO findings, batch-3)

After v1.4.2 shipped, a third 3-way adversarial review (Codex `gpt-5.5`
xhigh + Pi `deepseek-v4-pro:xhigh` + Cursor Composer 2) of the
**remaining** v1.4.0 SDK content -- the JS / Python / Go / Rust /
Java / .NET / PHP sections in `rules/sdks.md` that were NOT in the
v1.4.0 batch but were generated by the same model in earlier passes
-- returned **3/3 NO-GO** with the same wholesale-hallucination
failure mode. Direct upstream verification (`repo1.maven.org`,
PyPI, npm, Packagist, GitHub raw, NuGet) confirmed the drift.

#### `rules/sdks.md` Java SDK section (full rewrite)
- Corrected Maven version: latest is `2.0.1` (verified 2026-04-28
  via `repo1.maven.org/maven2/com/surrealdb/surrealdb/maven-metadata.xml`,
  not `3.0.0` as previously documented and not `1.0.0-beta.1` as the
  v1.4.2 Kotlin section had said based on a stale Maven Central
  solrsearch result).
- Replaced fabricated API surface (`db.connect("ws://...")`,
  `db.signin("root", "root")`, `db.use("ns", "db")`,
  `db.create("person", Map.of(...))`, `db.queryAsync(...)` returning
  `CompletableFuture<...>`) with the verified upstream API: typed
  `Credential` objects (`RootCredential`, `NamespaceCredential`,
  `DatabaseCredential`, `RecordCredential`, `BearerCredential`),
  chained `useNs(ns).useDb(db)`, typed generics on
  `create(Class<T>, table, value)` and `select(Class<T>, table)`,
  separate `query(sql)` / `queryBind(sql, params)` methods. Removed
  `queryAsync` / `CompletableFuture` (does not exist in source).
- Documented the verified embedded `memory` connection mode plus
  Java 8+ requirement and native-arch list.

#### `rules/sdks.md` PHP SDK section
- Corrected `signin()` keys: upstream uses `"user"` / `"pass"`, not
  `"username"` / `"password"` (silent auth failure with the wrong
  keys).
- Captured the `$token = $db->signin([...])` return value (signin
  returns a token string).
- Switched the SurrealQL `query()` example from a double-quoted
  string (which would interpolate `$min_age` as a PHP variable) to
  single-quoted.
- Replaced string record-IDs with the canonical typed
  `RecordId::create("person", "alice")` and `Table::create("person")`
  per upstream README.

#### `rules/sdks.md` Python SDK section
- Corrected embedded URL schemes: verified upstream
  `examples/embedded/` and `async_embedded.py` source comment pin
  the surface to `mem://` and `file://`. Replaced the previous
  `memory`, `surrealkv://`, and `rocksdb://` examples (none of
  which are documented in current upstream).

#### `rules/sdks.md` Go SDK section
- Removed the fabricated embedded URL schemes (`mem://`,
  `surrealkv://`). Upstream `db.go` documents only WebSocket and
  HTTP connection engines via the `New(url)` entry point; that
  entry point itself is marked `Deprecated` in favor of
  `FromEndpointURLString(ctx, url)`.
- Corrected method names and signatures: `SignIn(ctx, any)` (capital
  `I`, not `Signin`), `Use(ctx, ns, db) error`, `Close(ctx) error`
  (Close takes context). Switched the `Auth` argument from a
  pointer (`&surrealdb.Auth{...}`) to a value, matching the
  upstream comment examples.

#### `rules/sdks.md` SDK Selection Guide matrix
- Java embedded engine: changed from `No` to
  `Yes (memory only)`. Upstream README explicitly claims "Support
  of 'memory' (embedded SurrealDB)" and the getting-started example
  uses `driver.connect("memory")`.
- .NET embedded engine: changed from `No` to
  `Yes (SurrealDb.Embedded.* packages)`. Upstream
  `surrealdb/surrealdb.net` ships
  `SurrealDb.Embedded.InMemory`, `SurrealDb.Embedded.RocksDb`, and
  `SurrealDb.Embedded.SurrealKv` packages.
- Python embedded engine: refined to `Yes (mem:// / file://)` per
  current upstream examples.
- "When to Use Each SDK" Java entry rewritten with verified Maven
  pin (`com.surrealdb:surrealdb 2.0.1`) and embedded support note.

#### `rules/sdks.md` Kotlin section (carryover correction)
- Updated the in-section reference to the published Java SDK fallback
  from `1.0.0-beta.1` to `2.0.1` (the v1.4.2 entry was based on a
  stale Maven Central solrsearch result; `repo1.maven.org`
  metadata is authoritative).

#### `CHANGELOG.md` v1.4.2 entry
- Updated the inline mention of the Java SDK fallback version from
  `1.0.0-beta.1` to `2.0.1` with a note that the v1.4.2 narrative
  was based on a stale solrsearch result and is corrected here.

### Security posture
- No new scripts, binaries, or third-party network endpoints. All
  upstream verification was via public read-only APIs (PyPI,
  rubygems.org, repo1.maven.org, search.maven.org, NuGet,
  Packagist, raw GitHub). No new credential surface.
- Removing the wrong PHP signin keys closes a silent-auth-failure
  surface where a developer copy-pasting from v1.4.2 docs would
  build code that passes type checks, hits the wire, and gets back
  an unauthenticated session.

### Migration
No consumer code changes. Rule-file content has been replaced;
consumers that copy-pasted from v1.4.0 / v1.4.1 / v1.4.2 should
re-pin to v1.4.3 and re-derive any code from the corrected rule
text. In particular: bump Java Maven `<version>` from any of the
older pins (`3.0.0`, `1.0.0-beta.1`) to `2.0.1`; switch the Java
API to the typed `Credential` + `Class<T>` shape; switch PHP
`signin()` keys to `"user"` / `"pass"`; switch Python embedded URLs
to `mem://` / `file://`; switch Go to `FromEndpointURLString` +
`SignIn` + `Close(ctx)`.

## [1.4.2] - 2026-05-05

### Fixed (atomic-protocol patch — adversarial-review NO-GO findings, batch-2)

After v1.4.1 shipped, a follow-up 3-way adversarial review (Codex `gpt-5.5`
xhigh + Pi `deepseek-v4-pro:xhigh` + Cursor Composer 2) of the
**other** v1.4.0 additions -- the Swift / Kotlin / Ruby SDK sections in
`rules/sdks.md` and the `setup-surreal` section in `rules/deployment.md`
-- returned **3/3 NO-GO** with the same wholesale-hallucination failure
mode. Direct upstream verification (PyPI / RubyGems / Maven Central /
GitHub APIs / raw `Package.swift`+`build.gradle.kts`+`gemspec`+`action.yml`
on 2026-05-05) confirmed the drift. This patch shrinks the affected
sections to verified-only content; full API documentation for the
pre-release SDKs is deferred to v1.5.0.

#### `rules/deployment.md` `setup-surreal` section
- The repository `surrealdb/setup-surreal` is a **GitHub Action** (latest tag `v2.0.1`, 2024-12-13) for running SurrealDB inside CI workflows. It is **not** a CLI bootstrap binary.
- Removed all CLI install commands (`brew install surrealdb/tap/setup-surreal`, `cargo install setup-surreal`, `npx @surrealdb/setup-surreal` -- none exist; verified via crates.io / npm registry / Homebrew).
- Removed the fabricated subcommand surface (`init`, `upgrade`, `provision`, `grant`, `helm-values`, `verify`).
- Removed the fabricated TLS-mode flag set, scoped-user provisioning, Helm values export, systemd / launchd / Docker scaffolding tree, and integration table with this skill's `scripts/onboard.py` / `scripts/doctor.py`.
- Replaced with a concise GitHub Action usage block grounded in the upstream `action.yml` (verified inputs: `surrealdb_version`, `surrealdb_port`, `surrealdb_username`, `surrealdb_password`, `surrealdb_auth`, `surrealdb_strict`, `surrealdb_log`, `surrealdb_additional_args`, `surrealdb_retry_count`).

#### `rules/sdks.md` Swift section
- Corrected platform deployment targets: actual upstream `Package.swift` declares iOS 17+, macOS 14+, tvOS 17+, watchOS 10+, visionOS 1+ (the v1.4.0 documentation said iOS 16+, macOS 13+, tvOS 16+, watchOS 9+, visionOS 1+).
- Removed the `from: "1.0.0"` SwiftPM pin: the upstream repo has **no git tags** at the v1.4.2 cut. Pin `branch: "main"` only for development.
- Removed the false claim that `SurrealKit` (the Rust / TypeScript schema toolkit) bundles the Swift client. The two are independent dependencies.
- Removed the entire fabricated single-`Surreal()`-class API (`db.connect`, `db.signin(.root(...))`, `db.live(table:)`, `event.value()`, `event.recordID`, `db.on(.disconnected)`). Verified actual API uses two `actor` clients (`SurrealHTTPClient` and `SurrealWebSocketClient`), a `SignInCredentials` enum, `SurrealModel`-conforming typed values, freestanding macros (`#select`, `#create`, `#update`, `#delete`, `#live`), `SurrealPredicate`, `LiveEvent<T>` with `.decoded` + `.action` (`LiveAction` enum), and `AsyncStream` live queries.
- Detailed API examples deferred to v1.5.0 after upstream publishes a tagged release.

#### `rules/sdks.md` Kotlin section
- Removed Maven coordinates `com.surrealdb:surrealdb-kotlin:0.4.0`: Maven Central has no `surrealdb-kotlin` artifact, and the upstream `gradle.properties` declares `VERSION_NAME=0.1.0-SNAPSHOT`. (The published Java SDK is `com.surrealdb:surrealdb 2.0.1` -- v1.4.2 originally said `1.0.0-beta.1` based on a stale Maven Central solrsearch result; the authoritative `repo1.maven.org` metadata shows `latest=2.0.1` from 2026-04-28. Corrected in v1.4.3.)
- Corrected dep versions to upstream `build.gradle.kts`: Kotlin `2.1.10`, coroutines `1.10.1`, kotlinx-serialization `1.8.0`.
- Corrected KMP targets to verified set: `androidTarget()`, `jvm()`, `iosX64()`, `iosArm64()`, `iosSimulatorArm64()` -- no JS, no non-Apple Native target.
- Removed the fabricated `Surreal()` + `db.connect("rocksdb://...")` / `db.connect("mem://")` embedded-engine claim. The actual `SurrealClientConfig` only takes `httpEndpoint` and `wsEndpoint` strings.
- Removed the fabricated `query<Person>(...): List<Person>` generics, `@JvmOverloads` / `selectBlocking` Java-interop story, and `Flow`-returning live queries. Verified API uses `SurrealClient(config: SurrealClientConfig)`, `JsonElement` returns, `LiveQuerySubscription`, and a `SurrealAuthInput` sealed interface.

#### `rules/sdks.md` Ruby section
- Corrected version pin: latest gem `surrealdb` is `0.7.0` (published 2026-04-01 by SurrealDB authors). The v1.4.0 `~> 1.0` pin would not resolve.
- Corrected required Ruby: `>= 3.2` (verified from `surrealdb.gemspec`); the v1.4.0 documentation said 3.1+.
- Removed the entirely-fabricated `surrealdb-rails` gem, `SurrealDB::Record` ActiveRecord-shaped class, and `where`/`order`/`limit` chain examples. Neither the gem nor a `surrealdb/surrealdb-rails` GitHub repo exists (verified via rubygems.org and api.github.com).
- Corrected the `surrealdb-embedded` companion gem claim. The gem **does** exist on RubyGems at v0.7.0 (published 2026-04-01 by SurrealDB authors, FFI to `libsurrealdb_c`, supports `mem://` / `surrealkv://` / `file://` URLs); the v1.4.0 API surface descriptions for it were hallucinated, so the section now points to the gem with a "API documentation pending v1.5.0 verification" caveat rather than restating the fabricated shape.
- Corrected constructor: `SurrealDB::Client.new(url, **options)` then `.connect` (URL goes to constructor, not `connect`). Auth `signin(credentials_hash)` takes a positional Hash, not keyword arguments.
- Corrected live-query shape: `live(resource)` returns a UUID; subscribe with `db.subscribe(uuid) { |event| ... }` and clean up with `db.kill(uuid)`. The v1.4.0 enumerator-returning `live(...).each do |event|` shape does not exist.

#### `rules/sdks.md` SDK Selection Guide matrix
- Added a "Published release" row showing Swift = No (no tags), Kotlin = No (SNAPSHOT), Ruby = Yes (0.7.0), Java = Yes (beta).
- Marked Swift / Kotlin / Ruby embedded-engine claims as Unverified / No / Unverified (the v1.4.0 matrix incorrectly said all three shipped embedded engines).
- Reframed live-query rows to match verified shapes: Swift `AsyncStream`, Kotlin `LiveQuerySubscription`, Ruby UUID + subscribe.
- Reframed "When to Use Each SDK" entries for Kotlin / Swift / Ruby with v1.4.2 reality: Swift no published tag, Kotlin no Maven release, Ruby gem 0.7.0 (no Rails adapter).

#### Entry-point file syncs
- `AGENTS.md`: deployment.md descriptor reframed; skill version table 1.4.1 -> 1.4.2; onboard.py-agent example version bumped.
- `README.md`: `setup-surreal` capability blurb reframed as GitHub Action; deployment.md descriptor reframed; root version badge bumped.
- `SKILL.md`: `setup-surreal` ecosystem entry reframed; deployment.md descriptor reframed; metadata version bumped.
- `scripts/onboard.py`: deployment.md / langchain.md topics reframed; `new_project` / `ml_inference` / `agent_integration` / `editor_setup` decision trees reframed to v1.4.1+v1.4.2 reality.

#### `SOURCES.json` pins corrected
- `surrealdb/setup-surreal` -> `v2.0.1 (GitHub Action; not a CLI bootstrap)` (date 2024-12-13).
- `surrealdb/surrealdb.swift` -> `no published tag (pre-release; pin branch=main only)`.
- `surrealdb/surrealdb.kotlin` -> `0.1.0-SNAPSHOT (no Maven Central release at v1.4.2 cut)`.
- `surrealdb/surrealdb.rb` -> `0.7.0 (RubyGems surrealdb)` (date 2026-04-01).

### Security posture
- No new scripts, binaries, or third-party network endpoints. All upstream verification was via public read-only APIs (rubygems.org, search.maven.org, api.github.com, raw.githubusercontent.com, pypi.org, crates.io). No new credential surface.
- Removing the fabricated install commands closes a supply-chain risk surface: `brew install surrealdb/tap/setup-surreal`, `cargo install setup-surreal`, and `npx @surrealdb/setup-surreal` would 404 today, but a squatted package at any of those names would have been a vector if a developer copy-pasted from the v1.4.0 / v1.4.1 documentation.

### Migration
No consumer code changes. Rule-file content has been replaced; consumers
that copy-pasted from v1.4.0 / v1.4.1 should re-pin to v1.4.2 and
re-derive any code from the corrected rule text. In particular: drop
`surrealdb-rails` gem references (the gem does not exist), keep
`surrealdb-embedded` gem references but discard any v1.4.0 API
example for it (the gem is real at v0.7.0 but the documented API
shape was fabricated), drop `com.surrealdb:surrealdb-kotlin` Maven
coordinates (use the published Java SDK `com.surrealdb:surrealdb
1.0.0-beta.1` from Kotlin until the KMP package publishes), and drop
any `setup-surreal init / provision / grant` CLI invocations.

## [1.4.1] - 2026-05-05

### Fixed (atomic-protocol patch — adversarial-review NO-GO findings)

A 3-way adversarial review (Codex `gpt-5.5` xhigh + Pi `deepseek-v4-pro:xhigh`
+ Cursor Composer 2; Gemini 3.1 Pro quota-exhausted upstream) of the four
new rule files added in v1.4.0 returned **3/3 NO-GO**. Direct upstream
verification (PyPI, crates.io, npm, GitHub raw READMEs, surrealdb.com docs)
confirmed wholesale drift between the v1.4.0 documentation and current
upstream reality.

This patch shrinks each affected rule to verified-only content with explicit
"pending verification, deferred to v1.5.0" notes for unverified surfaces. No
new content is asserted that has not been read directly from upstream
sources fetched on 2026-05-05.

#### `rules/surrealmcp.md` — rewritten from upstream `README.md`
- Removed `cargo install surrealmcp` and `npm install -g @surrealdb/surrealmcp` (neither exists; crates.io 404, npm 404). Replaced with `cargo install --path .` and Docker.
- Replaced bare `surrealmcp` and `surrealmcp serve` invocations with the verified `surrealmcp start` subcommand.
- Replaced `--namespace` / `--database` / `--bind` / `--auth-token` with verified `--ns` / `--db` / `--bind-address` / `--access-token` (plus `--refresh-token`) flags. The `SURREAL_MCP_CLOUD_*` env-var names retain the `CLOUD_` infix; the CLI flags do not.
- Replaced env-var convention (`SURREAL_USER` / `SURREAL_PASS`) with the upstream-documented `SURREALDB_USER` / `SURREALDB_PASS` / `SURREALDB_URL` / `SURREALDB_NS` / `SURREALDB_DB` plus `SURREAL_MCP_*` server-side prefix.
- Replaced the hallucinated tool catalog (`merge`, `live`, `kill`, `schema.introspect`, `schema.tables`, `schema.table`, `info.db`, `info.ns`, `use`, `signin`) with the upstream-grouped tools: `query`, `select`, `insert`, `create`, `upsert`, `update`, `delete`, `relate`, `connect_endpoint`, `use_namespace`, `use_database`, `list_namespaces`, `list_databases`, `disconnect_endpoint`, plus cloud tools.
- Replaced `surrealmcp ping` with `curl http://localhost:8000/health`.
- Replaced `--max-concurrent-tools` with `--rate-limit-rps` / `--rate-limit-burst`. Replaced `--log-format json` with `RUST_LOG`.

#### `skills/surrealmcp/SKILL.md` — reconciled with the rule
- Updated quick-start, host-config, env vars, and tool catalog to match the rewritten rule. Reconciled `mcpServers` shape across rule and sub-skill.

#### `rules/langchain.md` — rewritten from upstream `README.md` + PyPI
- Removed entire JavaScript / TypeScript section. The `@langchain/surrealdb` npm package does not exist (registry 404).
- Removed `AsyncSurrealDBVectorStore`, `SurrealChatMessageHistory`, and `SurrealHybridRetriever` classes (none exist upstream).
- Replaced `from_endpoint()` / `from_client()` factory methods with the verified `SurrealDBVectorStore(embeddings, conn)` constructor.
- Corrected dependency claims: `langchain-core ~= 1.1.0` and `surrealdb ~= 1.0.8` (v1 SDK, not v2). Python `>= 3.10, < 4.0`.
- Corrected pip extras: at v0.2.1 the upstream `pyproject.toml` declares **no** `[project.optional-dependencies]`, so `[openai]`, `[huggingface]`, and `[graph-qa]` are all silent no-ops in pip. The README mentions a `[graph-qa]` extra (depends on `langchain-classic`) but the package metadata does not ship it; install `langchain-classic` explicitly until upstream wires the extra.
- Replaced `filter=` kwarg with the verified `custom_filter=`.

#### `rules/surrealml.md` — shrunk to scope-summary; v1.4.0 claims retracted
- Removed all `DEFINE MODEL` / `INFO FOR MODEL` / `REMOVE MODEL` SurrealQL claims. The SurrealDB v3 `DEFINE` statement list (verified at `https://surrealdb.com/docs/surrealql/statements/define`) contains ACCESS, ANALYZER, API, BUCKET, CONFIG, DATABASE, EVENT, FIELD, FUNCTION, INDEX, MODULE, NAMESPACE, PARAM, SCOPE, SEQUENCE, TABLE, TOKEN, USER — there is no `MODEL`.
- Removed `ml::name<version>(...)` invocation form (depends on the non-existent `DEFINE MODEL`).
- Removed `surreal start --user-mem-limit` flag (not in upstream CLI).
- Removed `surreal ml import --name --version` flags (the docs `/cli/ml/import` page 404s; treat the `surreal ml` surface as unstable).
- Removed Python `SurMlFile.from_pytorch / from_onnx / from_sklearn / from_keras / from_hf` factories and the `ModelMeta` class. The actual `surrealml 0.0.4` package uses an `Engine` enum + builder methods on `SurMlFile`.
- Corrected pip extras to the verified `[sklearn]`, `[torch]`, `[tensorflow]`. There is no `[hf]` extra.
- Fixed `DEFINE EVENT` example: `$value` -> `$after.id`. Standard event variables in SurrealDB are `$before`, `$after`, `$event`.

#### `rules/editor-tooling.md` — shrunk to verified-pointer summary
- Both `surrealql-language-server` (v0.1.2 on crates.io, 2026-04-21) and `surql-lsp` (v0.1.1 on crates.io, 2026-03-28) are real; the rule no longer asserts which is canonical.
- Removed the v1.4.0 VS Code command palette (`SurrealDB: Run Selection`, etc.) and settings catalog (`surrealdb.connections`, `surrealdb.activeConnection`, `surrealdb.auth.source`) — the published extension's `package.json` had zero of these registered.
- Removed the unverified `surrealql.toml` config-schema block.
- Removed the unverified `surrealql-language-server lint --format github` CI subcommand and the unverified `--socket <port>` flag.
- Trimmed editor-extension descriptions to discoverability pointers; per-editor command/setting detail is deferred to v1.5.0 after a manual upstream pass per editor.

#### `SOURCES.json` — version pins corrected
- Updated `surrealdb/surrealmcp.release` from `v0.2.0` to `v0.4.0` (verified via `api.github.com/repos/surrealdb/surrealmcp/releases`; tags v0.1.0 through v0.4.0 published 2025-08-21 to 2025-09-05).
- Updated `surrealdb/surrealml.release` from `v0.5.x` to `0.0.4 (PyPI surrealml)`.
- Updated `surrealdb/langchain-surrealdb.release` from `current` to `0.2.1 (PyPI langchain-surrealdb)`.

#### `.github/workflows/release.yml`
- Added `workflow_dispatch` trigger with a `tag` input so an existing release tag can be re-published without the draft-toggle dance. Wired through `actions/checkout` ref, `RELEASE_VERSION`, and the clawhub publish step.

### Security posture
- No new scripts, binaries, or third-party network endpoints. All upstream verification was via `curl` to public APIs (crates.io, PyPI, npm registry, GitHub raw, surrealdb.com docs). No new credential surface.
- The rule rewrites *reduce* the project's exposure: removing fabricated install commands eliminates the user-instruction failure mode where a developer attempts a non-existent `cargo install` or `npm install` (those would 404 today, but a newly-squatted package at one of those names would have been a supply-chain risk). All install paths now resolve to the upstream `surrealdb` GitHub org or Docker Hub.

### Migration
No consumer code changes (the skill ships rules + scripts; no library API). One CI workflow change: `.github/workflows/release.yml` gained a `workflow_dispatch` trigger so existing release tags can be re-published without the draft-toggle dance. Rule-file content has been replaced; consumers that copy-pasted from v1.4.0 should re-pin to v1.4.1 and re-derive any code from the corrected rule text. The `skills/surrealmcp/SKILL.md` quick-start has changed shape — update any host MCP config to use the verified env-var names (`SURREALDB_*`) and the `surrealmcp start` subcommand.

## [1.4.0] - 2026-05-03

> **Note (2026-05-05):** this release contained substantial drift from
> the actual upstream APIs across multiple new sections:
>
> - **`rules/surrealmcp.md`, `rules/editor-tooling.md`, `rules/langchain.md`,
>   `rules/surrealml.md`** -- hallucinated install commands, CLI flags,
>   env-var names, tool catalogs, SurrealQL syntax, Python class names,
>   and pip extras. **Retracted in v1.4.1.**
> - **`rules/sdks.md`** Swift / Kotlin / Ruby SDK sections -- hallucinated
>   Maven coordinates, version pins, platform deployment targets, embedded
>   engine support, API surfaces (`Surreal()` class shape, `db.connect`,
>   `db.live(table:)`, `event.value()`), and companion gems
>   (`surrealdb-rails`; `surrealdb-embedded` is real but the v1.4.0
>   API documentation for it was hallucinated). **Retracted in
>   v1.4.2.**
> - **`rules/deployment.md`** `setup-surreal` section -- documented the
>   project as an opinionated CLI bootstrap binary with `init` /
>   `provision` / `grant` / `helm-values` / `verify` subcommands and
>   `brew` / `cargo` / `npx` install paths. The actual repository is a
>   GitHub Action for CI (`uses: surrealdb/setup-surreal@v2`).
>   **Retracted in v1.4.2.**
>
> Adversarial review found the drift on the next pass; v1.4.1 + v1.4.2
> ship verified-only rewrites grounded in upstream READMEs and package
> registries. **Read the v1.4.2 and v1.4.1 entries above for the
> corrected surfaces; do not copy-paste from the v1.4.0 capability
> description that follows.**

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
