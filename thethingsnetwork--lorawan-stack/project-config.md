---
trigger: always_on
description: The Things Stack for LoRaWAN. Backend is Go; the web frontends (Console, Account App) are React. Development tooling is driven by [Mage](https://magefile.org/) via `tools/bin/mage`.
---

# The Things Stack v3

The Things Stack for LoRaWAN. Backend is Go; the web frontends (Console, Account App) are React. Development tooling is driven by [Mage](https://magefile.org/) via `tools/bin/mage`.

This file distills [CONTRIBUTING.md](CONTRIBUTING.md) and [DEVELOPMENT.md](DEVELOPMENT.md). Consult those for full details if in doubt. In case there is any mismatch with this file, the other files take precedence.

## Setup and common commands

```bash
make init                          # initialize tooling and dependencies (slow; run once, and after tooling changes)

# Run a development stack
tools/bin/mage js:build            # build frontend assets into public/ (slow, takes minutes)
tools/bin/mage dev:dbStart         # start databases in Docker (dev:dbStop also exists; dev:dbErase DESTROYS local data)
tools/bin/mage dev:initStack       # create DB, migrate, create admin/admin user
go run ./cmd/ttn-lw-stack -c ./config/stack/ttn-lw-stack.yml start   # Console at http://localhost:1885/

# Interactive frontend dev environment (stack + webpack-dev-server on :8080)
tools/bin/mage dev:serveDevWebui

# Tests
tools/bin/mage go:test             # full Go suite (slow — prefer targeted runs below)
tools/bin/mage js:test             # frontend unit tests (Jest over pkg/webui)
tools/bin/mage jsSDK:test          # JS SDK unit tests
go test ./pkg/<pkg>/...                # single package (preferred for targeted runs)
go test -run TestName ./pkg/<pkg>/...  # single Go test
node_modules/.bin/jest pkg/webui/<path-to-spec>  # single frontend test file
tools/bin/mage js:cypressHeadless  # end-to-end tests (see DEVELOPMENT.md for setup)
node_modules/.bin/cypress run --config-file config/cypress.config.js --spec <spec.js>  # single e2e spec

# Lint and format
golangci-lint run ./pkg/...        # Go lint (config in .golangci.yml)
golangci-lint run --new-from-rev=HEAD~1 ./<pkg_path>/... --fix # lint and fix only what has changed since the last commit
tools/bin/mage js:lint             # JS lint (eslint, config in config/eslintrc.yaml)
tools/bin/mage js:fmt              # JS format (prettier)

# Code generation — run after changing the corresponding sources, CI fails otherwise
tools/bin/mage proto:clean proto:all jsSDK:definitions   # after editing .proto files in api/ (needs Docker, slow)
tools/bin/mage go:messages         # after adding/changing errors, events or enums (updates config/messages.json)
tools/bin/mage js:translations     # after adding/changing frontend react-intl messages (updates pkg/webui/locales)
tools/bin/mage go:eventData        # after adding/changing events
```

Go tests that need Redis **skip themselves silently** unless `TEST_REDIS=1` is set and the databases are running (`dev:dbStart`) — passing output may mean skipped, not tested. `REDIS_ADDRESS`/`REDIS_DB` override the target instance; `TEST_SLOWDOWN` scales test timeouts on slow machines.

Add `-v` to mage for verbose output when a target fails silently. If a build is in a broken state, rebuild the frontend from scratch: `tools/bin/mage jsSDK:cleanDeps jsSDK:clean js:cleanDeps js:clean js:build`.

## Project structure

- `api/` — protocol buffer definitions (the source of truth for the API)
- `cmd/` — binaries: `ttn-lw-stack` and `ttn-lw-cli`
- `pkg/` — all Go libraries; one package per component (`networkserver`, `applicationserver`, `identityserver`, ...)
  - `pkg/ttnpb/` — generated code from protos (do not edit by hand)
  - `pkg/webui/` — frontend: `console/` and `account/` apps, shared `components/`, `containers/`, `lib/`, `locales/`
- `config/stack/` — development stack configuration files
- `cypress/` — frontend end-to-end tests
- `sdk/js/` — JavaScript SDK
- `data/` — data from external repositories (devices, frequency plans, webhook templates)
- `tools/` — Mage-based dev/test/build tooling
- `public/`, `release/` — build output, not committed

## Generated and vendored files — do not edit by hand

| Path                                        | Regenerate with                                          |
| ------------------------------------------- | -------------------------------------------------------- |
| `pkg/ttnpb/` and other proto-generated code | `tools/bin/mage proto:clean proto:all jsSDK:definitions` |
| `config/messages.json`                      | `tools/bin/mage go:messages`                             |
| `pkg/webui/locales/*.js`                    | `tools/bin/mage js:translations`                         |
| `data/`                                     | pulled from external repositories — do not edit          |

If CI complains about one of these files, fix the source (proto, error/event definition, message definition, `go.mod`) and regenerate — never patch the generated file directly.

## Go code style

- Formatting via `gofmt`/`goimports`; tabs for indentation (enforced by `.editorconfig`). Lint with `golangci-lint`.
- Line length: prefer a natural break past 80 columns, break lines past 120.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TheThingsNetwork/lorawan-stack](https://github.com/TheThingsNetwork/lorawan-stack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
