---
trigger: always_on
description: A Rust CLI/TUI for browsing AI models, benchmarks, and coding agents. Built with ratatui, crossterm, and tokio.
---

# Models — Claude Code Instructions

## Project Overview
A Rust CLI/TUI for browsing AI models, benchmarks, and coding agents. Built with ratatui, crossterm, and tokio.

## Build & Test
```bash
mise run fmt        # Format code (required before commit)
mise run clippy     # Lint with -D warnings
mise run test       # Run tests
mise run build      # Build debug
mise run run        # Run the TUI
```

Always run the full check sequence before committing:
```bash
mise run fmt && mise run clippy && mise run test
```

## Architecture

### Tabs
- **Models Tab** (`src/tui/models/`) — browse models from models.dev API with 3-column layout (20% providers | 45% model list | 35% detail panel), RTFO capability indicators, adaptive provider panel
- **Benchmarks Tab** (`src/tui/benchmarks/`) — compare model benchmarks from Artificial Analysis with browse/compare modes, H2H table, scatter plot, radar chart views
- **Agents Tab** (`src/tui/agents/`) — track AI coding assistants with GitHub integration
- **Status Tab** (`src/tui/status/`) — live provider health monitoring with detail view for incidents, components, and scheduled maintenance

### Data Flow
- Model data: fetched from models.dev API at startup (`src/api.rs`)
- Benchmark data: fetched fresh from jsDelivr CDN on every launch (`src/benchmarks/fetch.rs`)
- Agent/GitHub data: disk-cached with ETag conditional fetching (`src/agents/cache.rs`, `src/agents/github.rs`)
- CLI agents: uses `fetch_releases_only` (1 API call, no repo metadata) — TUI uses full `fetch_conditional` (2 calls, includes stars/issues/license)
- Status data: fetched from each provider's official status page (Statuspage, BetterStack, Instatus, etc.) with apistatuscheck.com as fallback (`src/status/fetch.rs`), provider registry and strategy mapping in `src/status/registry.rs`
- Status source contract and normalization rules are documented in code comments within `src/status/` adapters

### Async Pattern
Background fetches use tokio::spawn + mpsc channels. Results arrive as `Message` variants processed in the main loop (`src/tui/mod.rs`). The app never blocks on network calls.

### Agents & CLI
See `src/agents/CLAUDE.md` and `src/cli/CLAUDE.md` for detailed module docs.
- Binary aliases: `models agents <cmd>` or `agents <cmd>` via argv[0] symlink detection. Alias names configurable via `[aliases]` in config.toml (defaults: `agents`, `benchmarks`, `mstatus`)
- Commands: `list`, `search`, `show`, `benchmarks`, `completions <shell>`, `link`, full agents suite (`status`, `latest`, `list-sources`, `<tool>`), full status suite (`list`, `show`, `status`, `sources`)
- CLI pickers use shared `PickerTerminal` infrastructure in `src/cli/picker.rs`

### Key Files

Each module has its own `CLAUDE.md` with detailed documentation. Top-level highlights:

- `src/formatting.rs` — shared utilities: `truncate`, `parse_date`, `format_tokens`, `format_stars`, `EM_DASH`, `cmp_opt_f64`
- `src/data.rs` — Provider/Model data structures from models.dev API
- `src/config.rs` — user config file (agents, cache, display, aliases settings). `AliasesConfig` struct + `AliasKind` enum for symlink routing
- `src/provider_category.rs` — provider categorization logic
- `src/benchmarks/` — `store.rs` (BenchmarkStore/Entry), `fetch.rs` (CDN fetcher), `traits.rs` (AA↔models.dev matching)
- `src/status/` — `types.rs`, `registry.rs`, `assessment.rs`, `fetch.rs`, `adapters/` (per-source-family parsers)
- `src/tui/` — `app.rs` (App state, Message enum), `event.rs` (NavAction dedup), `ui.rs` (shared helpers), `markdown.rs`, `widgets/` (ScrollablePanel, SoftCard, ScrollOffset, ComparisonLegend), per-tab subdirs: `models/`, `agents/`, `benchmarks/` (includes `radar.rs`), `status/` — each with `app.rs` (sub-app state) + `render.rs` (tab rendering)
- `src/agents/health.rs` — agent-to-status-provider mapping for service health display in the Agents tab
- `src/cli/` — `picker.rs` (shared PickerTerminal, nav helpers, style constants), `models.rs`/`benchmarks.rs`/`agents_ui.rs`/`status.rs` (inline pickers), `styles.rs`

### GitHub Actions
- `ci.yml` — runs on PR/push: fmt check, clippy, test
- `release.yml` — triggered by `v*` tags: builds 5 targets in parallel with Rust caching, packages .deb/.rpm via cargo-binstall (pinned versions), generates SHA256SUMS, publishes to crates.io, and updates AUR package. Homebrew Core updates are handled in `Homebrew/homebrew-core` by Homebrew automation/maintainers, not from this repo. Pre-release tags (containing `-`) skip publish/AUR and mark the GitHub release as prerelease. Scoop Extras handles Windows updates via its own autoupdate mechanism.
- `update-benchmarks.yml` — runs every 30 minutes: fetches AA API, commits if data changed

## Conventions
- Use `mise run <task>` for all CLI operations — never run bare commands
- Keep clippy clean with `-D warnings`
- Enum-based message passing (no callbacks)
- No disk cache — benchmark data fetched fresh from CDN on every launch, empty store until CDN responds
- `BenchmarkEntry` must derive both `Serialize` and `Deserialize`
- New `BenchmarkEntry` fields require `#[serde(default)]`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [reyamira/models](https://github.com/reyamira/models) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
