---
name: surrealism
description: "SurrealDB Surrealism WASM extension development. Write Rust functions, compile to WASM, deploy as database modules. Part of the surreal-skills collection."
license: MIT
metadata:
  version: "1.0.0"
  author: "24601"
  parent_skill: "surrealdb"
---

# Surrealism -- WASM Extensions for SurrealDB

Surrealism enables you to write custom functions in Rust, compile them to WebAssembly (WASM), and deploy them as native SurrealDB modules. This sub-skill provides guidance for building, testing, and deploying WASM extensions.

## Prerequisites

- Rust toolchain (stable) with `wasm32-unknown-unknown` target
- SurrealDB CLI v3+ (`surreal` binary)
- `wasm-pack` or `cargo` with WASM support
- Familiarity with SurrealDB's extension system

## Quick Start

```bash
# Create a new Surrealism project
cargo new --lib my_extension
cd my_extension

# Add the WASM target
rustup target add wasm32-unknown-unknown

# Build to WASM
cargo build --target wasm32-unknown-unknown --release

# Import into SurrealDB
surreal import --conn http://localhost:8000 --ns test --db test extension.wasm
```

## What You Can Build

- Custom scalar functions callable from SurrealQL
- Data transformation pipelines
- Domain-specific logic (validation, computation, encoding)
- Integration adapters for external formats

## Full Documentation

See the main skill's rule file for complete guidance:
- **[rules/surrealism.md](../../rules/surrealism.md)** -- project setup, Rust function signatures, WASM compilation, deployment, testing, and best practices
