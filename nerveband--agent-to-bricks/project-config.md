---
trigger: always_on
description: Instructions for AI agents (Claude Code, Codex, etc.) working on this project.
---

# Agent to Bricks -- agent guide

Instructions for AI agents (Claude Code, Codex, etc.) working on this project.

## Project structure

```
agent-to-bricks/
  cli/              Go CLI (cobra, bubbletea TUI)
  gui/              Tauri desktop app
  plugin/           WordPress plugin (PHP)
  website/          Website / marketing
  docs/             Documentation and reference data
    plans/          Design docs and implementation plans
    test-data/      452 Frames templates (gitignored)
    agents/         Agent reference data (gitignored)
  tests/            Test suites
    snippets/       JS browser snippet tests
  schema/           JSON schema definitions
  scripts/          Deploy and build scripts
  Makefile          build / test / install / deploy-staging
```

## Build and test

```bash
make build          # builds bin/bricks
make install        # installs the CLI into the preferred local bin dir
make install-verify # build/install verification + staging connectivity when configured
make test           # runs all Go tests
make test-verbose   # verbose test output
make lint           # go vet
make check-version  # sync/version drift check, including cli/schema.json
make staging-gate   # full staging deploy + plugin/CLI/template/GUI E2E gate
```

The CLI binary is at `bin/bricks` after build. Tests live next to the code they test (`_test.go` files).

Staging and local install automation are env-driven. Load repo-local `.env` when present and treat `.env.example` as the source of truth for required `ATB_STAGING_*` variables, WP-CLI deploy settings, and optional compatibility aliases.

`docs/test-data/` may contain proprietary/local-only Bricks fixtures and is intentionally gitignored. Public clones must skip private corpus validation cleanly when those fixtures are absent.

## Core philosophy

Treat these as non-optional design constraints on every subsequent code change:

- ShipTypes-style contract-first interfaces: public CLI/plugin/GUI surfaces should be typed, machine-readable, and driven by one canonical contract instead of prose-only behavior.
- Agent DX CLI discipline: preserve or improve structured JSON I/O, raw payload input, schema validation, stable error shapes, pagination/field-selection controls, safety rails, and automation-friendly defaults.
- No silent corruption of structured content: avoid regex or naive string splitting that rewrites valid CSS, HTML, JSON, URLs, or other structured payloads unless the parser is delimiter-aware.
- Docs and implementation move together: when behavior, contract, or workflow changes, update tests, docs, prompts, and `AGENTS.md` in the same change.

If a change would make the product less machine-addressable, less contract-driven, or more likely to damage structured content in transit, treat that as a regression and fix the design before merging.

## CLI architecture

Go module at `cli/`. Entry point is `cli/main.go`. Commands are in `cli/cmd/`. Core libraries are in `cli/internal/`.

Key packages:
- `internal/client/` -- REST API client for the WordPress plugin
- `internal/convert/` -- HTML-to-Bricks converter, class registry, style parser
- `internal/agent/` -- LLM context builder (md/json/prompt output)
- `internal/templates/` -- template catalog, Frames format loader, composer
- `internal/config/` -- YAML config at `~/.agent-to-bricks/config.yaml`

## Plugin architecture

PHP plugin at `plugin/agent-to-bricks/`. REST API namespace: `agent-bricks/v1`.

Auth: `X-ATB-Key` header with API key (generated in WP Admin > Settings > Agent to Bricks).

Key classes:
- `class-elements-api.php` -- GET/PUT/PATCH/DELETE page elements
- `class-classes-api.php` -- global class CRUD
- `class-snapshots-api.php` -- snapshot and rollback
- `class-site-api.php` -- site info, framework detection
- `class-convert-api.php` -- HTML-to-Bricks conversion (tag/style mapping, class resolution)
- `class-element-validator.php` -- element schema validation
- `class-api-auth.php` -- API key auth (hashed storage, X-ATB-Key header)

## Versioning

CLI and plugin share one semver. Both must show the same major.minor version.

- Canonical release version: `VERSION`
- CLI version: injected via ldflags at build time from `VERSION`
- Plugin version: `AGENT_BRICKS_VERSION` constant + `Version:` header in `agent-to-bricks.php`

Only bump versions for a release or when explicitly asked. Use `make sync-version` to propagate `VERSION` into generated/versioned assets before release.

## Release process

Each release is one atomic unit: CLI binaries + plugin zip, same version.

### Steps

1. Update `VERSION`, then run `make sync-version` so the plugin header/constant, docs, and `cli/schema.json` stay aligned.
2. Run the local gate before tagging:
   ```bash
   ./scripts/verify-local-install.sh
   make test
   make lint
   cd cli && go run . schema --validate
   cd gui && npx tsc --noEmit
   find plugin -name "*.php" -exec php -l {} \;
   cd website && npm run build
   ```
3. Run `./scripts/verify-staging-release.sh` when staging credentials are configured. This is the canonical release gate and covers staging deploy, plugin runner matrix, CLI E2E, template smoke, and GUI MCP E2E.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nerveband/agent-to-bricks](https://github.com/nerveband/agent-to-bricks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
