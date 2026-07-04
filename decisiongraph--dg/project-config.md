---
trigger: always_on
description: Use Rust for all tools built here. Read `objective.md` for project goals.
---

Use Rust for all tools built here. Read `objective.md` for project goals.

## External repos

- `decisiongraph/graphs-tui` — Terminal diagram renderer (D2/Mermaid → ASCII art). Published as `graphs-tui` crate on crates.io.

## Workspace crates

### crates/md-db — Core library
Markdown-as-database: YAML frontmatter parsing, KDL schema validation, document graph, discovery, search, diffing, migration, sync, export, suggestions, static site generation. Used by dg-cli and dg-mcp.

### crates/dg-cli — CLI binary (`dg`)
User-facing CLI. Depends on md-db for all document operations, markdown-tui for terminal rendering.

### crates/dg-mcp — MCP server (`dg-mcp`)
JSON-RPC over stdio. 11 tools: dg-validate, dg-get, dg-list, dg-inspect, dg-describe, dg-set, dg-new, dg-refs, dg-graph, dg-check-code, dg-deprecate.

### crates/dg-schemas — Built-in schemas & templates
Embeds KDL schema, Claude/Gemini/OpenCode templates, org.kdl template via `include_str!`. Pure data crate. Exports `ALL_TEMPLATES`, `SCHEMA`, `CLAUDE_MD`, skill templates, etc.

### crates/markdown-tui — Terminal markdown renderer
GFM → ANSI strings or ratatui widgets. Tables with box-drawing, syntax-highlighted code blocks, math, callouts, task lists. Also `render_table()` for data tables.

### crates/gherkin (dg-gherkin) — Gherkin parser
Parse & validate Gherkin scenarios from SPEC documents. Generate D2/Mermaid diagrams.

### cc-eval/ — Claude Code eval runner (standalone, not in workspace)

## Code paths by change scenario

### Changing the static site (`dg site` / `dg export --site`)

The site is generated as an mdbook-style static HTML site with sidebar, search, prev/next navigation.

**Entry points:**
- `dg-cli/src/commands/site.rs` — `dg site` command (args, roadmap HTML building)
- `dg-cli/src/commands/export.rs` — `dg export --site` (delegates to `site.rs` helpers)

**Core generation (all in `md-db/src/site/`):**
- `mod.rs` — `generate_site()` orchestrator: discovers docs, builds graph, groups by type, collects pages, wraps in layout, writes to disk. Returns page count.
- `pages.rs` — Page generators: `intro_page()` (README→index), `onboarding_page()`, `doc_list_page()`, `doc_page()`, `team_pages()`, `user_pages()`, `org_pages()`, `graph_page()`, `roadmap_page()`
- `layout.rs` — `render_page_layout()` wraps body+sidebar into full HTML
- `nav.rs` — `build_nav_tree()`, `render_sidebar_html()`, `flat_page_order()`, `prev_next_links()`
- `css.rs` — All CSS as a constant
- `js.rs` — All JS as a constant (search, sidebar toggle, theme)
- `search.rs` — `generate_search_index()` → JSON for client-side search

**Supporting modules used by site:**
- `md-db/src/export.rs` — `render_markdown_to_html()`, `frontmatter_meta()`, `linkify_refs()`, CSS/JS constants, `GRAPH_JS`
- `md-db/src/graph.rs` — `DocGraph::build()` for backlinks
- `md-db/src/discovery.rs` — `discover_files()` for finding documents
- `md-db/src/roadmap.rs` — `build_roadmap()`, `render_roadmap_html()` for roadmap page
- `md-db/src/users.rs` — `OrgConfig` for team/user/org pages

**Config struct** (`md-db/src/site/mod.rs`):
```rust
SiteConfig { title, roadmap: bool, users: bool, roadmap_html: Option<String> }
```

### Changing document types or schema

**Schema definition:** `crates/dg-schemas/schema.kdl` (embedded at compile time)

**Schema parser:** `md-db/src/schema.rs`
- `Schema { types, relations, ref_formats }`
- `TypeDef { name, aliases, folder, fields, sections, rules, preamble, singleton, match_pattern }`
- `FieldDef { name, field_type, required, pattern, default }`
- `SectionDef { name, required, children, table, content, list, diagram }` (recursive)
- `RuleDef { when_field, when_equals, then_required, then_section_table }` (conditional)

**Ripple effects of schema changes:**
- `md-db/src/validation.rs` — Validates docs against schema (diagnostic codes: F0xx frontmatter, S0xx structure, C0xx content, R0xx refs, L0xx preamble, SG0xx singletons)
- `md-db/src/template.rs` — `generate_document()` creates new docs from schema definition
- `md-db/src/suggest.rs` — Uses schema for optional section/diagram checks
- `md-db/src/site/pages.rs` — Type map hardcoded: `[("adr","architecture"), ("opp","opportunities"), ("pol","policies"), ("inc","incidents"), ("spec","specifications")]`
- `md-db/src/site/nav.rs` — Nav tree built from type groups
- `dg-mcp/src/tools.rs` — `dg-describe` exposes schema to AI agents
- `dg-schemas/` — Template skills reference type names

### Changing validation logic

- `md-db/src/validation.rs` — Main validation engine. Entry: `validate_document()`, `validate_directory()`
- `md-db/src/schema.rs` — Schema rules (conditional `when`/`then`, cardinality)
- `md-db/src/graph.rs` — `find_dangling_refs()`, `find_cycles()`, `find_orphans()` for graph-level checks
- `dg-cli/src/commands/validate.rs` — CLI wrapper
- `dg-cli/src/commands/lint.rs` — validate + graph health combined
- `dg-mcp/src/tools.rs` — `dg-validate` tool

### Changing CLI commands

**Pattern:** Each command is a module in `dg-cli/src/commands/`. To add:
1. Create `commands/mycommand.rs` with `pub struct MyArgs` (clap `Args`) + `pub fn run(...)`
2. Add `pub mod mycommand;` to `commands/mod.rs`
3. Add `MyCommand(mycommand::MyArgs)` variant to `Command` enum
4. Add match arm in `main.rs`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [decisiongraph/dg](https://github.com/decisiongraph/dg) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
