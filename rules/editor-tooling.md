# SurrealQL Editor Tooling

> **v1.4.1 status note:** the v1.4.0 version of this rule documented a
> `surrealql.toml` config schema, a feature matrix, a CI `lint`
> subcommand, a `--socket` flag, a VS Code command palette, and a
> long settings catalog -- none of which were verified against
> upstream. The v1.4.0 version also asserted official SurrealDB
> editor extensions for editors where the actual extension is
> community-maintained or whose published surface differs from
> what was documented. This file has been shrunk to verified content
> only; full detail is deferred to v1.5.0 after a manual upstream
> pass per editor.

This rule covers the editor-side toolchain for SurrealQL: language
servers, tree-sitter grammar, and editor extensions. Pair this with
`rules/surrealmcp.md` for the agent-facing layer -- LSP serves human
authoring, MCP serves agent execution.

---

## Verified Components (2026-05-05)

| Component | Verified status | Notes |
|-----------|-----------------|-------|
| `surrealql-language-server` (crates.io) | v0.1.2 published 2026-04-21 | Newer of the two LSP crates; description: "Language Server Protocol implementation for SurrealQL" |
| `surql-lsp` (crates.io) | v0.1.1 published 2026-03-28 | Earlier LSP crate; same intent, different name |
| `surrealql-tree-sitter` (GitHub) | Repo exists | README content not verified for this revision |

**Both LSP crates are real.** Pick one for your editor wiring; we have
not verified which is canonical or whether one supersedes the other.
When deciding, check upstream commit activity and any guidance from
the SurrealDB team in the surrealdb GitHub org.

---

## Editor Extensions: Status Reality Check

The following surfaces existed at the v1.4.1 cut but the **detail of
what each exposes** (commands, settings, config schemas, key
bindings) was **not verified upstream** for this rule revision.
Treat the entries below as discoverability pointers; consult each
extension's own README before relying on any specific feature.

| Editor | Where to look |
|--------|---------------|
| VS Code / Cursor / Windsurf / VSCodium | VS Code Marketplace + OpenVSX, search `SurrealQL` |
| JetBrains (IntelliJ, RustRover, GoLand, PyCharm, WebStorm, RubyMine) | JetBrains Marketplace, search `SurrealQL` (note: upstream plugin name has been seen as `surql-jetbrains`; settings location on disk has varied) |
| Neovim | `surrealdb/surrealql-neovim` (verify on the surrealdb GitHub org) |
| Helix | Configure via `languages.toml` once an LSP binary is on `$PATH` |
| Zed | Zed extensions panel; community extensions exist |
| Sublime Text | Wire via the LSP package + a SurrealQL syntax package |
| Emacs | `surrealdb/surrealql-emacs` (verify on the surrealdb GitHub org) |

Specifically retracted from v1.4.0:

- **The VS Code command palette** (`SurrealDB: Run Selection`,
  `SurrealDB: Run File`, `SurrealDB: Connect`, `SurrealDB: Open Schema
  Browser`, `SurrealDB: Open in Surrealist`) -- these were not
  verified in the published extension's `package.json`. Do not assume
  any of them exists; check the extension's own contributions list.
- **The settings catalog** (`surrealdb.connections`,
  `surrealdb.activeConnection`, `surrealdb.auth.source`) -- not
  verified. Use the extension's documented settings only.
- **The `surrealql.toml` config schema** -- not verified against any
  built LSP. Configure via the LSP's actual documented settings only.
- **`surrealql-language-server lint --format github` CI command** --
  not verified. Use whichever lint surface upstream actually exposes.
- **`--socket <port>` flag on the LSP** -- not verified.

---

## Minimal LSP Wiring Examples

These wire the LSP through your editor without asserting any specific
config schema. The shape is whatever the LSP itself accepts; consult
the upstream README for `surrealql-language-server` (or the LSP crate
you chose).

### Neovim (`nvim-lspconfig`)

```lua
require("lspconfig").surrealql.setup({
  cmd = { "surrealql-language-server" },
  filetypes = { "surrealql" },
  root_dir = require("lspconfig.util").root_pattern(".git"),
})
```

### Helix (`languages.toml`)

```toml
[[language]]
name = "surrealql"
scope = "source.surrealql"
file-types = ["surql"]
language-servers = ["surrealql-language-server"]

[language-server.surrealql-language-server]
command = "surrealql-language-server"
```

### VS Code / Cursor / Windsurf

```bash
# Marketplace
code --install-extension surrealdb.surrealql

# OpenVSX (VSCodium, code-server)
codium --install-extension surrealdb.surrealql
```

> The publisher ID `surrealdb.surrealql` was reported by both
> reviewers; verify it on the marketplace before scripting.

---

## Choosing Between LSP, MCP, and Surrealist

These three surfaces overlap but optimize for different audiences:

| Audience | Primary surface | Why |
|----------|-----------------|-----|
| Developer authoring SurrealQL in their editor | LSP + extension | Inline diagnostics, completion, formatting -- editor-native |
| Coding agent (Claude, Cursor, Copilot, Codex) | SurrealMCP | Stable tool catalog, structured introspection, no UI |
| Operator running ad-hoc queries / debugging | Surrealist | Visual schema designer, query history, graph visualizer |

A typical setup runs all three: VS Code with the LSP for human work,
the MCP server registered in your AI host config for agent loops,
and Surrealist on the side for ops.

---

## Cross-References

- `rules/surrealql.md` -- the language the LSP and grammar parse
- `rules/surrealmcp.md` -- agent-facing equivalent
- `rules/surrealist.md` -- standalone GUI / IDE
- `rules/surrealkit.md` -- desired-state schema files the LSP would lint
- `references/surrealql_cheatsheet.md` -- quick syntax reference
- `surrealql-language-server` on crates.io: `https://crates.io/crates/surrealql-language-server`
- `surql-lsp` on crates.io: `https://crates.io/crates/surql-lsp`
