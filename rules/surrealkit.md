# SurrealKit -- Schema Sync, Rollouts, Seeds, and Declarative Tests

SurrealKit is a schema management tool for SurrealDB applications. It treats
`.surql` files as the desired state, reconciles disposable databases directly,
and uses rollout manifests for reviewed staged changes on shared or production
databases.

Tracked upstream snapshot: **v0.5.0** (`b5c22f745a4e`, 2026-04-10).

> **Packaging note**: the current README in `surrealdb/surrealkit` still points
> to historical `ForetagInc/surrealkit` release binaries while the active repo
> is now `surrealdb/surrealkit`. Treat Cargo installation as the stable path
> unless the release distribution story is clarified upstream.

---

## Installation

```bash
cargo install surrealkit
```

## Project Layout

```text
database/
  schema/       # desired-state .surql definitions
  rollouts/     # planned rollout manifests (.toml)
  snapshots/    # local catalog/schema snapshots
  seeds/        # seed files
  tests/
    config.toml
    suites/*.toml
```

Initialize the scaffold:

```bash
surrealkit init
```

## Environment Variables

SurrealKit reads these values into its local `.env` workflow:

- `DATABASE_HOST` or `PUBLIC_DATABASE_HOST`
- `DATABASE_NAME` or `PUBLIC_DATABASE_NAME`
- `DATABASE_NAMESPACE` or `PUBLIC_DATABASE_NAMESPACE`
- `DATABASE_USER`
- `DATABASE_PASSWORD`

## Operating Modes

### Sync

Use `sync` when the database can match local files immediately.

```bash
surrealkit sync
surrealkit sync --watch
```

Behavior:

- Applies changed definitions from `database/schema`
- Removes SurrealKit-managed objects deleted from desired state
- Best for local, preview, CI, and other disposable databases

For shared databases, destructive prune requires an explicit override:

```bash
surrealkit sync --allow-shared-prune
```

### Rollouts

Use rollouts for shared or production databases where changes need review,
staging, rollback, or operator-controlled cutover.

```bash
surrealkit rollout baseline
surrealkit rollout plan --name add_customer_indexes
surrealkit rollout start 20260302153045__add_customer_indexes
surrealkit rollout complete 20260302153045__add_customer_indexes
surrealkit rollout rollback 20260302153045__add_customer_indexes
surrealkit rollout lint 20260302153045__add_customer_indexes
surrealkit rollout status
```

Workflow:

1. Author desired state in `database/schema/*.surql`
2. Baseline the shared database once
3. Generate a rollout manifest
4. Start the non-destructive expansion phase
5. Cut application traffic/code over
6. Complete the contract/destructive phase
7. Roll back if the rollout is still in-flight and the cutover fails

Generated manifests are stored in `database/rollouts/*.toml`.

## Seeding

Run seed data explicitly:

```bash
surrealkit seed
```

Use seeds for deterministic local fixtures, smoke datasets, and test setup. Do
not rely on seeds as a substitute for rollout-managed schema changes.

## Declarative Testing

Run declarative suites:

```bash
surrealkit test
surrealkit test --json-out test-results.json
```

Suite files live under `database/tests/suites/*.toml`.

Supported case kinds:

- `sql_expect`
- `permissions_matrix`
- `schema_metadata`
- `schema_behavior`
- `api_request`

Useful flags:

- `--suite <glob>`
- `--case <glob>`
- `--tag <tag>` (repeatable)
- `--fail-fast`
- `--parallel <N>`
- `--json-out <path>`
- `--no-setup`
- `--no-sync`
- `--no-seed`
- `--base-url <url>`
- `--timeout-ms <ms>`
- `--keep-db`

By default, suites run in isolated ephemeral namespace/database pairs and fail
CI on any test failure.

## Decision Guidance

- Use `sync` for disposable environments where desired-state deletion is safe.
- Use `rollout` for shared or production databases, especially when dropping or
  renaming fields, indexes, access definitions, or tables.
- Use `seed` for fixtures and example data, not schema evolution.
- Use `test` to validate permissions, schema invariants, and API behavior in CI
  before rollout completion.

## Common Patterns

### Local Development Loop

```bash
surrealkit sync --watch
surrealkit seed
surrealkit test --tag smoke
```

### Shared Environment Change

```bash
surrealkit rollout plan --name add_order_indexes
surrealkit rollout lint 20260410120000__add_order_indexes
surrealkit rollout start 20260410120000__add_order_indexes
# deploy app cutover
surrealkit rollout complete 20260410120000__add_order_indexes
```

### CI Validation

```bash
surrealkit sync
surrealkit seed
surrealkit test --fail-fast --json-out artifacts/surrealkit-tests.json
```
