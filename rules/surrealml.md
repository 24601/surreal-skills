# SurrealML -- Machine Learning Models in SurrealDB

> **v1.4.1 status note:** the v1.4.0 version of this rule documented a
> `DEFINE MODEL ml::name<version>(...)` SurrealQL syntax, an
> `INFO FOR MODEL` statement, a `REMOVE MODEL` statement, an
> `ml::name<version>(...)` invocation form, a `surreal start
> --user-mem-limit` flag, an `surreal ml import` CLI invocation,
> a `db.upload_ml(...)` SDK method, and a Python SDK with
> `SurMlFile.from_pytorch / from_onnx / from_sklearn / from_keras /
> from_hf` factories plus a `ModelMeta` class. **None of those
> currently exist upstream** -- the SurrealDB v3 `DEFINE` statement
> list contains ACCESS, ANALYZER, API, BUCKET, CONFIG, DATABASE,
> EVENT, FIELD, FUNCTION, INDEX, MODULE, NAMESPACE, PARAM, SCOPE,
> SEQUENCE, TABLE, TOKEN, USER -- no MODEL. The `surrealml` PyPI
> package (v0.0.4) is early-stage with a different API surface than
> what was documented. This file has been shrunk to a scope summary;
> a full rewrite grounded in the actual `surrealdb/surrealml` source
> is deferred to v1.5.0.

SurrealML is the SurrealDB project for storing and serving machine
learning models. The package compiles to a `.surml` artifact format
that bundles an ONNX graph plus metadata, intended to be loadable
into a SurrealDB server and invokable from queries.

- Upstream: `https://github.com/surrealdb/surrealml`
- PyPI: `https://pypi.org/project/surrealml/` (v0.0.4 at the v1.4.1 cut)
- Status: early-stage / preview -- API and SurrealQL surface are not
  stable; treat anything below as a pointer, not a contract

---

## Current Reality (verified 2026-05-05)

- The PyPI package `surrealml 0.0.4` exists. Its extras are
  `[sklearn]`, `[torch]`, `[tensorflow]` -- there is no `[hf]` extra.
  Pinned deps include `numpy==1.26.3`.
- The package's Python API uses an `Engine` enum and builder methods
  on a `SurMlFile` (verified against the upstream `clients/python`
  source); it does **not** expose the `from_pytorch / from_onnx /
  from_sklearn / from_keras / from_hf` factory methods that v1.4.0
  documented.
- The SurrealDB `DEFINE` statement list does **not** include `MODEL`.
  Any rule, plan, or example that uses `DEFINE MODEL`, `INFO FOR
  MODEL`, `REMOVE MODEL`, or an `ml::name<version>(...)` call site is
  not portable to current upstream. Verify against
  `https://surrealdb.com/docs/surrealql/statements/define` before
  copying any such snippet.
- The `surreal ml` CLI root page exists at
  `https://surrealdb.com/docs/surrealdb/cli/ml`, but `/import` and
  `/export` subpages 404 at the v1.4.1 cut. Treat the CLI surface as
  a moving target.

---

## When to Reach for SurrealML

If your data lives in SurrealDB and you want a model to score it
without leaving the database -- yes. If you serve a single model
behind a high-QPS HTTP API with no DB coupling, keep the model in a
dedicated serving runtime (Triton, Ray Serve, BentoML).

The promise is *colocation*: model + features + downstream record
in the same engine.

---

## Working API Touchpoints

These are the parts the upstream README/PyPI confirm exist; the
exact shape is what `surrealml/clients/python` exposes -- consult it
before writing code, and pin to a specific upstream commit:

- The `.surml` artifact format (Rust core in the upstream repo,
  consumed by both the Python and Rust clients).
- Optional dependency groups for the Python client: `[sklearn]`,
  `[torch]`, `[tensorflow]`.

> Anything beyond these surfaces (SurrealQL invocation, CLI import,
> server-side permissions on a model) is unstable enough to warrant
> direct upstream verification rather than copy-paste from this
> rule.

---

## Composing With Other SurrealDB Surfaces

The patterns below are stable and don't depend on `DEFINE MODEL`:

### Vector indexes for embeddings

```surql
DEFINE TABLE document SCHEMAFULL;
DEFINE FIELD content   ON TABLE document TYPE string;
DEFINE FIELD embedding ON TABLE document TYPE array<float>;
DEFINE INDEX idx_doc_embedding ON TABLE document FIELDS embedding
  HNSW DIMENSION 384 DIST COSINE;
```

For the supported vector-index types and dimensions, see
`rules/vector-search.md`. MTREE was removed in v3.

### Computing embeddings server-side via DEFINE EVENT

If your eventual goal is to invoke a model server-side from an
event, today the practical path is:

1. Compute the embedding client-side (or via a SurrealDB function
   you've defined).
2. `UPDATE` the record with the embedding in the same transaction
   that created it.

The standard event-variable convention in SurrealDB is `$before`,
`$after`, and `$event`. Use `$after` (or `$after.id`) when writing
back to the record, not `$value`.

```surql
DEFINE EVENT compute_embedding ON TABLE document WHEN $event = "CREATE" THEN {
  UPDATE $after.id SET embedding = fn::compute_embedding($after.content);
};
```

`fn::compute_embedding` here is a placeholder for whatever surface
ends up exposing your model -- a Surrealism extension, a stable
SurrealML invocation form once the upstream API lands, or a
client-side write before insert.

---

## Cross-References

- `rules/vector-search.md` -- HNSW, dimensions, distance metrics
- `rules/data-modeling.md` -- computed fields, events, schemafull tables
- `rules/surrealism.md` -- Rust-based extensions (composes with SurrealML once it stabilizes)
- `rules/surrealkit.md` -- rollout-managed schema upgrades
- `rules/security.md` -- record-level permissions
- `rules/langchain.md` -- LangChain pipelines that produce embeddings
- Upstream: `https://github.com/surrealdb/surrealml`
- Upstream docs portal: `https://surrealdb.com/docs/`
