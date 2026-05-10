---
trigger: always_on
description: CiteBox is a Go + SQLite application with a native HTML/CSS/JavaScript frontend. Main entrypoints live in `cmd/server` and `cmd/desktop`. Core backend layers are under `internal/`: `handler` (HTTP), `service` (business logic), `repository` (SQLite), `model`, `config`, and shared app wiring in `internal/app`. Frontend pages are in `web/`, with shared assets in `web/static/`. Packaging scripts live in `scripts/`, build targets in `Makefile`, and longer-form docs in `docs/`.
---

# Repository Guidelines

## Project Structure & Module Organization

CiteBox is a Go + SQLite application with a native HTML/CSS/JavaScript frontend. Main entrypoints live in `cmd/server` and `cmd/desktop`. Core backend layers are under `internal/`: `handler` (HTTP), `service` (business logic), `repository` (SQLite), `model`, `config`, and shared app wiring in `internal/app`. Frontend pages are in `web/`, with shared assets in `web/static/`. Packaging scripts live in `scripts/`, build targets in `Makefile`, and longer-form docs in `docs/`.

## Build, Test, and Development Commands

- `make run`: start the web server at `http://localhost:8080`.
- `make run-desktop`: launch the desktop client with embedded local server.
- `make build` / `make build-desktop`: compile server or desktop binaries into `bin/`.
- `make test`: run the full Go test suite (`go test ./...`).
- `make prepare-web-assets`: fetch `pdf.js` runtime assets required by some web flows.
- `make package-desktop-linux|darwin|windows`: create desktop distribution archives in `dist/`.
- `scripts/macos-desktop-ui-smoke.zsh smoke`: macOS-only desktop UI smoke test for close prompt, "minimize to tray", and Dock reopen flows. Start the desktop app first, and make sure Terminal/`osascript` already has Accessibility permission.

For targeted macOS desktop UI checks, `scripts/macos-desktop-ui-smoke.zsh` also exposes helper commands: `processes`, `windows`, `tree`, `close-prompt`, `to-tray`, `dock-items`, and `dock-reopen`. Override the default process/window/Dock names with `CITEBOX_MACOS_PROCESS_NAME`, `CITEBOX_MACOS_WINDOW_NAME`, and `CITEBOX_MACOS_DOCK_ICON_NAME` when the local app label differs.

When editing frontend code, also syntax-check touched files, for example:

```bash
node --check web/static/js/library.js
```

## Coding Style & Naming Conventions

Use `gofmt` for all Go files; keep package names lowercase and exported identifiers in `PascalCase`. Follow the existing layered design: handlers should stay thin, services own workflow logic, repositories own SQL and migrations. Frontend code uses plain JavaScript objects/modules, 4-space indentation, `camelCase` identifiers, and descriptive `data-*` hooks in HTML. Prefer ASCII unless a file already contains localized copy.

## I18n Guidelines

- Treat i18n as required for all user-visible frontend copy. Do not add or keep new hardcoded UI text in HTML or JavaScript when it can be served through the existing locale system.
- When adding or changing frontend text, update both `web/static/locales/zh-CN/` and `web/static/locales/en/` in the same change, and keep keys aligned across languages.
- Put page-specific strings in the matching page locale file and reserve `shared.json` for copy reused across multiple pages or features.
- When wiring UI text, prefer the existing translation hooks and helpers already used in the repo, such as `data-i18n` attributes and `t(...)`/`CiteBoxI18n` lookups.

## Language Content Policy

- Do not introduce hardcoded natural-language content outside Chinese and English in code, tests, fixtures, configuration, docs, or locale files unless the user explicitly asks for that language-specific content.
- Do not add Japanese, Korean, or other non-Chinese/non-English examples just to exercise multilingual behavior. Use abstracted fixtures, transliteration-free markers, or provider/model mocks instead.
- Keep production routing, search, and prompt logic language-agnostic where possible. If language-specific behavior is required, make the supported language scope explicit and keep it limited to Chinese and English.

## Testing Guidelines

Go tests are colocated as `*_test.go` files, especially under `internal/repository` and `internal/service`. Name tests as `Test<Behavior>`. Add repository tests for schema changes, migrations, constraints, and search behavior. Use `go test ./...` before submitting; for UI-only changes, include at least a JS syntax check and a brief manual verification note.

When changing macOS desktop window-management flows, tray behavior, close-confirmation dialogs, or Dock reopen behavior, run `scripts/macos-desktop-ui-smoke.zsh smoke` as part of manual verification. If the full smoke command fails, use the script's helper subcommands to isolate whether the issue is process discovery, window selection, close prompt rendering, tray minimization, or Dock reopening.

## Commit & Pull Request Guidelines


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xuzhougeng/citebox](https://github.com/xuzhougeng/citebox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
