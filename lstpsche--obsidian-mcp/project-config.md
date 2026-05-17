---
trigger: always_on
description: > **Self-update rule:** If you learn something new about this project — a design decision,
---

# CLAUDE.md — Agent Context for obsidian-mcp

> **Self-update rule:** If you learn something new about this project — a design decision,
> a gotcha, a pattern that worked or failed, a dependency quirk, or any other durable fact —
> update this file before finishing your task. Keep entries concise and in the right section.
> Don't duplicate what's already here; amend or refine existing entries instead.

## What This Project Is

**obsidian-mcp** is a Rust MCP (Model Context Protocol) server that gives AI agents full
read/write access to an Obsidian vault through direct filesystem operations.

It replaces two existing projects:
- [mcp-obsidian](https://github.com/MarkusPfundstein/mcp-obsidian) — a Python MCP server that wraps the Obsidian REST API
- [obsidian-local-rest-api](https://github.com/coddingtonbear/obsidian-local-rest-api) — an Obsidian community plugin that exposes a local REST API

**Key difference:** This project talks directly to the vault filesystem. No Obsidian plugins,
no Obsidian running, no HTTP. Just a single static binary that reads/writes markdown files.

## Architecture

```
AI Client (Cursor / Claude Desktop / etc.)
    │ stdio (MCP JSON-RPC)
    ▼
obsidian-mcp binary
    ├── src/tools/       ← MCP tool handlers (rmcp #[tool_router])
    ├── src/vault/       ← vault layer (fs, parsing, index, watcher)
    ├── src/models.rs    ← shared types
    ├── src/config.rs    ← env/CLI config
    └── src/error.rs     ← unified VaultError
    │ filesystem
    ▼
~/vault/  (directory of .md files + .obsidian/ config)
```

### Data flow

1. AI client sends MCP tool call over stdio
2. `rmcp` deserializes into tool params, routes to handler
3. Handler calls `Vault` methods
4. `Vault` delegates to vault sub-modules (fs, parser, frontmatter, index, patch, periodic)
5. Result serialized back as MCP response

### Concurrency model

- The `Vault` struct wraps the index in `Arc<RwLock<VaultIndex>>`
- Tool handlers take `&self` (shared reference) — reads acquire read lock, writes acquire write lock
- The filesystem watcher runs in a background tokio task, updating the index on file changes
- The `Vault` is `Send + Sync + Clone` so rmcp can share it across async tasks

## Technology Stack

| Component | Crate | Purpose |
|-----------|-------|---------|
| MCP protocol | `rmcp` 1.6 | Server, tools, stdio + Streamable HTTP transport |
| HTTP framework | `axum` 0.8 | HTTP routing for Streamable HTTP transport |
| Async runtime | `tokio` 1 | async/await, tasks, IO |
| Serialization | `serde`, `serde_json`, `yaml_serde` 0.10 | JSON + YAML frontmatter |
| JSON Schema | `schemars` 1.0 | Auto-generate schemas for tool params |
| Markdown | `pulldown-cmark` 0.13 | Parse headings, detect code blocks |
| Filesystem | `walkdir` 2, `notify` 8, `globset` 0.4 | Walk dirs, watch changes, glob match |
| Regex | `regex` 1 | Wikilink/tag/block-ref extraction, search |
| Full-text search | `tantivy` 0.26 | BM25 inverted index, stemming, ranked search |
| Dates | `chrono` 0.4 | Periodic notes date handling |
| Errors | `thiserror` 2 | Derive error types |
| Logging | `tracing` 0.1, `tracing-subscriber` 0.3 | Structured logging |

## Obsidian Vault Format Reference

### File structure
- Vault = a directory on disk containing `.md` files
- `.obsidian/` at vault root holds app config (settings, plugin data, themes)
- Notes are plain UTF-8 markdown with optional YAML frontmatter

### Frontmatter (Properties)
```yaml
---
tags: [rust, mcp]
aliases: [obsidian-server]
date: 2026-03-19
custom_field: any value
---
```
- YAML between `---` delimiters at the very start of the file
- The `tags` and `aliases` fields have special meaning to Obsidian
- Values can be text, lists, numbers, booleans, dates

### Wikilinks
| Syntax | Meaning |
|--------|---------|
| `[[note]]` | Link to note (resolved by shortest unique path) |
| `[[note\|alias]]` | Link with display text |
| `[[note#heading]]` | Link to heading in note |
| `[[note#^blockid]]` | Link to block reference |
| `[[#heading]]` | Link to heading in current note |
| `![[note]]` | Embed (transclude) note content |

**Resolution:** Obsidian uses shortest-unique-path matching. `[[foo]]` resolves to the
only `foo.md` in the vault regardless of folder depth. If ambiguous, full path is needed.

### Tags
- Inline: `#tag`, `#nested/tag` — must start with a letter, case-insensitive
- Frontmatter: `tags: [tag1, tag2]`
- Cannot be purely numeric (`#123` is invalid, `#y123` is valid)

### Block references
- `^blockid` at the end of a line or on its own line
- IDs: alphanumeric + dashes
- Referenced via `[[note#^blockid]]`

### Periodic notes config
- Core daily notes: `.obsidian/daily-notes.json`
  ```json
  { "format": "YYYY-MM-DD", "folder": "Daily", "template": "Templates/Daily" }
  ```
- Periodic Notes plugin: `.obsidian/plugins/periodic-notes/data.json`
- Date format uses Moment.js tokens (YYYY, MM, DD, dddd, etc.)

## rmcp Patterns

### Tool definition
```rust
#[derive(Deserialize, JsonSchema, Default)]
struct MyParams {
    /// Description shown to the AI
    field: String,
}

#[tool(name = "tool_name", description = "What the tool does")]
async fn tool_name(
    &self,
    Parameters(params): Parameters<MyParams>,
) -> Result<CallToolResult, ErrorData> {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lstpsche/obsidian-mcp](https://github.com/lstpsche/obsidian-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
