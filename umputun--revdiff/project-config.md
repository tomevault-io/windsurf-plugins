---
trigger: always_on
description: TUI for reviewing diffs, files, and documents with inline annotations, built with bubbletea.
---

# revdiff

TUI for reviewing diffs, files, and documents with inline annotations, built with bubbletea.

**Architecture**: see [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) for system design, data flows, interfaces, and design decisions.

## Commands
- Build: `make build` (output: `.bin/revdiff`)
- Test: `make test` (race detector + coverage, excludes mocks)
- Lint: `make lint` or `golangci-lint run`
- Format: `make fmt` or `~/.claude/format.sh`
- Generate mocks: `go generate ./...`
- Vendor after adding deps: `go mod vendor`

## Project Structure
- `app/` - composition root (`package main`), split by concern: `main.go` (entrypoint + `run()`), `config.go` (options/parsing), `stdin.go` (stdin mode), `renderer_setup.go` (VCS wiring), `themes.go` (theme CLI + adapter), `history_save.go` (session save)
- `app/diff/` - VCS interaction (git + hg + jj), unified diff parsing, VCS detection, Mercurial + Jujutsu support
- `app/ui/` - bubbletea TUI package. Single `Model` struct with state grouped into sub-structs (`cfg`, `layout`, `file`, `modes`, `nav`, `search`, `annot`), methods split across files by concern (~500 lines each). Each source file has a matching `_test.go`. See `app/ui/doc.go` for package docs, `docs/ARCHITECTURE.md` for file-by-file breakdown. Does not import `app/theme` or `app/fsutil` — theme operations go through the `ThemeCatalog` interface
- `app/ui/style/` - color/style resolution: hex-to-ANSI, lipgloss styles, SGR tracking, HSL math. Types: `Resolver`, `Renderer`, `SGR`
- `app/ui/sidepane/` - file tree + markdown TOC components with cursor/offset management
- `app/ui/worddiff/` - intra-line word-diff: tokenizer, LCS, line pairing, highlight insertion
- `app/ui/overlay/` - layered popups: help, annotation list, theme selector. Manager enforces one-at-a-time
- `app/highlight/` - chroma syntax highlighting, foreground-only ANSI
- `app/keymap/` - configurable keybindings (`Action` constants, parser, defaults, dump)
- `app/theme/` - Catalog-centric theme system: `Theme` (data + serialization) and `Catalog` (discovery, loading, installation, gallery). Zero standalone functions — all logic as methods. Files: `theme.go` (Theme struct), `catalog.go` (Catalog struct + all operations). 7 bundled + community gallery
- `app/annotation/` - in-memory annotation store and structured output
- `app/editor/` - external `$EDITOR` invocation for multi-line annotations: temp-file lifecycle, editor resolution ($EDITOR → $VISUAL → vi), and `Command()` API returning `*exec.Cmd` + completion func for `tea.ExecProcess`. Consumed by `app/ui` via the `ExternalEditor` interface
- `app/history/` - review session auto-save to `~/.config/revdiff/history/`
- `app/fsutil/` - filesystem utilities
- `app/ui/mocks/` - moq-generated mocks (never edit manually)

## Architecture Principles
- **Decouple OS/external concerns from UI**: OS-level work (exec.Command, os.CreateTemp, env lookup, network calls) does not belong in `app/ui/`, even for small helpers. Extract to a dedicated package (e.g. `app/editor/`) even when there's only one production caller. The cost of a new package is lower than keeping `app/ui/` entangled with OS boundaries.
- **Minimize exported surface**: first pass of a new package almost always over-exports. Before finalizing, ask "which of these does the caller actually need?" and unexport the rest. For stateless helpers, prefer unexported methods on the grouping struct over top-level unexported functions — matches the global "prefer methods over standalone utilities" rule.
- **Consumer-side interfaces for external deps**: when UI uses an external subsystem, the default shape is: interface in `app/ui/` (consumer side), concrete type in the subsystem package, injection via `ModelConfig` (nil defaults to concrete). Direct import of a concrete type from an external package into `app/ui` is a smell — the interface documents the dependency direction and leaves room for alternate implementations, not just test mocks.

## Config
- Config file: `~/.config/revdiff/config` (INI format via go-flags IniParser)
- Precedence: CLI flags > env vars > config file > built-in defaults
- `--dump-config` outputs current defaults, `--config` overrides path
- `no-ini:"true"` tag excludes fields from config file (used for --config, --dump-config, --dump-theme, --list-themes, --init-themes, --version)
- Themes dir: `~/.config/revdiff/themes/` with 7 bundled themes, auto-created on first run
- `--theme NAME` loads theme; `--dump-theme` exports resolved colors; `--list-themes` lists available; `--init-themes` re-creates bundled
- Theme precedence: `--theme` overwrites all 23 color fields + chroma-style, ignoring `--color-*` flags or env vars
- Theme values applied via `applyTheme()` in `themes.go` which overwrites `opts.Colors.*` after `parseArgs()`. `colorFieldPtrs(opts)` is the single source of truth for color key → struct field mapping — adding a new color requires changes in `theme.go` colorKeys + options struct + `colorFieldPtrs()` in `themes.go`
- Theme ownership split: `app/theme.Catalog` owns discovery/loading, `app/ui.ThemeCatalog` interface consumed by UI, `app/themes.go` wires adapter composing catalog + config persistence
- `ini-name` tags ensure config keys match CLI long flag names

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [umputun/revdiff](https://github.com/umputun/revdiff) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->
