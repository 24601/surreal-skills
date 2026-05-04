# SurrealQL Editor Tooling

This rule covers the editor-side toolchain for SurrealQL: the language server
(LSP), the tree-sitter grammar, and the official editor extensions for Visual
Studio Code, JetBrains IDEs, Neovim, Helix, Sublime Text, and Zed. Pair this
with `rules/surrealmcp.md` for the agent-facing layer; the two surfaces are
complementary -- LSP handles human authoring, MCP handles agent execution.

---

## Components at a Glance

| Component | Repo | What it provides |
|-----------|------|------------------|
| `surrealql-language-server` | `surrealdb/surrealql-language-server` | Language Server Protocol implementation: completion, hover, diagnostics, go-to-definition, formatting, semantic tokens |
| `surrealql-tree-sitter` | `surrealdb/surrealql-tree-sitter` | Tree-sitter grammar used for syntax highlighting, structural navigation, and incremental parsing in modern editors |
| `surrealql-vsx` | `surrealdb/surrealql-vsx` | SurrealQL grammar definition for TextMate, VS Code, VSCodium, Cursor, Windsurf, and other IDEs that consume `.tmGrammar` |
| `surrealql-jetbrains` | `surrealdb/surrealql-jetbrains` | JetBrains plugin (IntelliJ IDEA, RustRover, GoLand, PyCharm, WebStorm, RubyMine) |
| `surrealql-neovim` | `surrealdb/surrealql-neovim` | Neovim plugin wiring the language server + tree-sitter grammar |
| `surrealql-helix` | `surrealdb/surrealql-helix` | Helix configuration / queries |
| `surrealql-zed` | `surrealdb/surrealql-zed` | Zed extension |
| `surrealql-emacs` | `surrealdb/surrealql-emacs` | Emacs major mode |
| `Surrealist` | `surrealdb/surrealist` | Standalone IDE (covered in `rules/surrealist.md`) |

The LSP and the tree-sitter grammar are the substrate; every editor extension
on this list is a thin wrapper that connects the editor's UI to those two
artifacts.

---

## Language Server (`surrealql-language-server`)

### Capabilities

| Feature | Status |
|---------|--------|
| Completion (statements, functions, table names from live DB) | Yes |
| Hover (function signatures, type info) | Yes |
| Diagnostics (parse + semantic) | Yes |
| Go-to-definition (DEFINE FIELD / FUNCTION / TABLE) | Yes |
| Find references | Yes |
| Document formatting | Yes |
| Range formatting | Yes |
| Rename (DEFINE-bound identifiers) | Yes |
| Semantic tokens | Yes |
| Document symbols | Yes |
| Workspace symbols | Yes |
| Code actions (auto-import field defs, fix typos) | Yes |
| Inline values during query debug | Experimental |

### Installation

```bash
# From source (Rust)
cargo install surrealql-language-server

# Verify
surrealql-language-server --version
```

The binary speaks LSP over stdio by default; `--socket <port>` is available
for editors that prefer TCP.

### Configuration

The LSP reads `surrealql.toml` from the workspace root (or `.surrealdb/lsp.toml`
if present). Common fields:

```toml
# Connect for live schema completion. Optional -- the LSP works statically
# from .surql files alone, but live connection unlocks completion for
# table/field names from a running DB.
[connection]
endpoint = "ws://localhost:8000/rpc"
namespace = "dev"
database = "main"
auth = "env"          # read SURREAL_USER / SURREAL_PASS

[format]
indent = 2
trailing_comma = true
keyword_case = "upper" # "upper" | "lower" | "preserve"

[lint]
unused_definitions = "warn"
shadowed_variables = "warn"
implicit_record_id = "info"

[paths]
# Glob patterns relative to workspace root
schema = ["database/schema/**/*.surql"]
rollouts = ["database/rollouts/**/*.toml"]
```

`auth = "env"` is the recommended posture -- the LSP never persists credentials
to disk and reads `SURREAL_USER` / `SURREAL_PASS` from the editor's environment
on demand.

### Diagnostics Severity

- **Error** -- parse failures, undefined fields against connected schema, type
  mismatches the planner would reject
- **Warning** -- shadowed variables, unused `LET`, missing `RETURN` in
  multi-statement blocks
- **Info** -- style hints (implicit IDs, redundant `FETCH`, deprecated function
  spellings)
- **Hint** -- replaceable tokens for code actions

---

## Tree-Sitter Grammar

The tree-sitter grammar provides incremental parsing used by syntax
highlighting, structural selection, and motion plugins.

```bash
# Use directly from any tree-sitter consumer
git clone https://github.com/surrealdb/surrealql-tree-sitter
cd surrealql-tree-sitter
npm install
npm run build
```

It produces a parser that is consumed by:

- Helix's built-in language registry (auto-discovered via `languages.toml`)
- Neovim's `nvim-treesitter` plugin (`:TSInstall surrealql`)
- Zed's extension system
- GitHub's Linguist (for syntax highlighting on github.com)

### Highlight Captures

The grammar exposes the standard tree-sitter highlight names used by editor
themes:

| Capture | Meaning |
|---------|---------|
| `keyword` | `SELECT`, `CREATE`, `RELATE`, `DEFINE`, `FOR`, `WHERE`, etc. |
| `keyword.type` | `string`, `int`, `bool`, `array`, `record`, `geometry`, etc. |
| `function.builtin` | `time::now`, `string::concat`, `vector::similarity::cosine`, etc. |
| `function` | User-defined function references |
| `variable.builtin` | `$auth`, `$session`, `$value`, `$this`, `$parent` |
| `string` / `string.escape` / `string.special` | String, escape, regex |
| `number` / `number.float` | Literals |
| `constant.builtin` | `NONE`, `NULL`, `true`, `false` |
| `operator` | `->`, `<-`, `<->`, `<\|K,EF\|>`, `?:`, etc. |
| `comment` | Line + block |
| `tag` | Future blocks (`<future>`) |

---

## VS Code / Cursor / Windsurf / VSCodium

The official extension is published to both the VS Code Marketplace and
OpenVSX, which means it works in:

- VS Code
- VSCodium
- Cursor
- Windsurf
- code-server (browser-hosted VS Code)

### Install

```bash
# Marketplace (VS Code, Cursor, Windsurf)
code --install-extension surrealdb.surrealql

# OpenVSX (VSCodium, code-server)
codium --install-extension surrealdb.surrealql
```

Or search **SurrealQL** in the Extensions tab.

### Settings (`settings.json`)

```jsonc
{
  // Treat .surql files as SurrealQL
  "files.associations": {
    "*.surql": "surrealql"
  },

  // Connect for live schema completion
  "surrealdb.connections": [
    {
      "name": "local",
      "endpoint": "ws://localhost:8000/rpc",
      "namespace": "dev",
      "database": "main"
    }
  ],
  "surrealdb.activeConnection": "local",

  // Authenticate from env (preferred). Alternatives: "vault" (1Password,
  // Bitwarden via the system credential helper), "prompt" (interactive)
  "surrealdb.auth.source": "env",

  // Format on save
  "[surrealql]": {
    "editor.formatOnSave": true,
    "editor.defaultFormatter": "surrealdb.surrealql"
  }
}
```

### Commands

| Command | Default keybinding | Effect |
|---------|--------------------|--------|
| `SurrealDB: Run Selection` | `Ctrl/Cmd+Shift+Enter` | Run highlighted SurrealQL against active connection |
| `SurrealDB: Run File` | -- | Execute the entire file |
| `SurrealDB: Connect` | -- | Pick / change the active connection |
| `SurrealDB: Open Schema Browser` | -- | Read-only schema tree from live DB |
| `SurrealDB: Open in Surrealist` | -- | Open the current connection in Surrealist |

---

## JetBrains (IntelliJ, RustRover, GoLand, PyCharm, WebStorm, RubyMine)

Install **SurrealQL** from the JetBrains Marketplace
(`Settings -> Plugins -> Marketplace -> SurrealQL`).

The plugin provides:

- Syntax highlighting + structural navigation backed by the LSP
- Live schema completion when a Data Source is configured (uses the JetBrains
  database connector or a project-level `surrealql.toml`)
- Inspections that surface LSP diagnostics in the editor gutter
- Run gutter icons per statement (matching IntelliJ's SQL plugin UX)
- File templates for `database/schema/*.surql` and rollout manifests

Configure connections in `Settings -> Languages & Frameworks -> SurrealQL ->
Connections`. Credentials are stored in the JetBrains password safe; the
plugin never writes them to project files.

---

## Neovim

Install via your plugin manager. The plugin orchestrates the LSP + tree-sitter
grammar and registers a few user commands.

### lazy.nvim

```lua
{
  "surrealdb/surrealql-neovim",
  dependencies = {
    "neovim/nvim-lspconfig",
    "nvim-treesitter/nvim-treesitter",
  },
  ft = { "surrealql" },
  opts = {
    lsp = {
      cmd = { "surrealql-language-server" },
      settings = {
        connection = {
          endpoint = "ws://localhost:8000/rpc",
          auth = "env",
        },
      },
    },
  },
}
```

After install:

```vim
:TSInstall surrealql
:LspInfo                 " confirm surrealql-language-server is attached
:checkhealth surrealql   " plugin self-check
```

### Manual `nvim-lspconfig`

If you prefer not to use the plugin:

```lua
require("lspconfig").surrealql.setup({
  cmd = { "surrealql-language-server" },
  filetypes = { "surrealql" },
  root_dir = require("lspconfig.util").root_pattern("surrealql.toml", ".git"),
})
```

### Helix

Helix discovers the grammar and LSP automatically once both are on PATH.
`languages.toml`:

```toml
[[language]]
name = "surrealql"
scope = "source.surrealql"
file-types = ["surql"]
language-servers = ["surrealql-language-server"]
indent = { tab-width = 2, unit = "  " }

[language-server.surrealql-language-server]
command = "surrealql-language-server"
```

Run `hx --grammar fetch && hx --grammar build` after pointing Helix at the
grammar repo.

---

## Zed

Install via the Zed extensions panel (`zed: extensions -> SurrealQL -> Install`).
The Zed extension wraps `surrealql-language-server` and the tree-sitter grammar.

`~/.config/zed/settings.json`:

```jsonc
{
  "languages": {
    "SurrealQL": {
      "format_on_save": "on",
      "tab_size": 2
    }
  },
  "lsp": {
    "surrealql-language-server": {
      "settings": {
        "connection": {
          "endpoint": "ws://localhost:8000/rpc",
          "auth": "env"
        }
      }
    }
  }
}
```

---

## Sublime Text

A community LSP-Sublime client wraps the official `surrealql-language-server`. Install:

1. Install **LSP** from Package Control
2. Install the **SurrealQL** syntax package (provides the tree-sitter binding
   and file association)
3. Add to `LSP.sublime-settings`:

```jsonc
{
  "clients": {
    "surrealql": {
      "enabled": true,
      "command": ["surrealql-language-server"],
      "selector": "source.surrealql"
    }
  }
}
```

---

## Choosing Between LSP, MCP, and Surrealist

These three surfaces overlap but optimize for different audiences:

| Audience | Primary surface | Why |
|----------|-----------------|-----|
| Developer authoring SurrealQL in their editor | LSP + extension | Inline diagnostics, completion, formatting -- all editor-native |
| Coding agent (Claude Code, Cursor agent, Codex) | SurrealMCP | Stable tool catalog, structured introspection, no UI |
| Operator running ad-hoc queries / debugging | Surrealist | Visual schema designer, query history, graph visualizer |

A typical setup runs all three: VS Code with the LSP for human work, the MCP
server registered in `~/.claude/mcp.json` for agent loops, and Surrealist on
the side for operations.

---

## CI Integration

Use the LSP / tree-sitter stack in CI to catch broken `.surql` files before
they hit the database.

```bash
# Install once
cargo install surrealql-language-server

# Run as a one-shot lint over the schema dir
surrealql-language-server lint database/schema --format github
```

The `--format github` option emits GitHub Actions annotations directly. For
JetBrains TeamCity / GitLab Pipelines, use `--format json` and post-process
with `jq`.

---

## Cross-References

- `rules/surrealql.md` -- the language the LSP and grammar parse
- `rules/surrealmcp.md` -- agent-facing equivalent of the LSP surface
- `rules/surrealist.md` -- standalone GUI / IDE
- `rules/surrealkit.md` -- desired-state schema files that the LSP lints
- `references/surrealql_cheatsheet.md` -- quick syntax reference
