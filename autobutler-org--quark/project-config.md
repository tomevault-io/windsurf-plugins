---
trigger: always_on
description: These instructions tell coding agents how to work in this repository.
---

# `AGENTS.md`

## Purpose

These instructions tell coding agents how to work in this repository.
[`CLAUDE.md`](CLAUDE.md) imports this file, so this is the one place to edit — never write a second copy of a
rule somewhere else.

Quark is a self-hosted personal cloud: a Go/Gin backend serving a Flutter client (web, iOS, Android) from a
device in the user's home, with SQLite holding the little state that cannot live on disk. Both halves live in
this repository.

### Repo map

```text
cmd/quark/                    entrypoint: install, serve, version
internal/db/                  sqlc-generated queries and golang-migrate migrations
internal/server/routes.go     where every router is mounted
internal/server/api/v0/<x>/   HTTP handlers, one directory per URL segment
internal/server/middleware/   gin middleware (auth, admin)
internal/server/public/       the embedded web build (//go:embed), generated
pkg/util/<x>util/             the business logic the handlers call
pkg/vfs/                      the virtual filesystem every file access goes through
pkg/backup/, pkg/calendar/    standalone services
sql/queries/                  sqlc query sources
lib/                          the Flutter app
packages/                     independent Dart/Flutter packages (see Packages Directory)
test/                         Flutter tests; Go tests sit beside the code they cover
docs/                         contributor documentation
scripts/                      CI and layout-check scripts
datalinks/                    symlinks to system directories, an in-repo view of app data
```

### Where to read next

- [`docs/dev-onboarding.md`](docs/dev-onboarding.md) — running it locally, the two backend modes, `AS_ROOT=1`.
- [`docs/user-journeys/`](docs/user-journeys/README.md) — ten files of stable `JN-XXX` journeys. This is the
  feature inventory; check it before claiming Quark does something.
- [`lib/widgets/README.md`](lib/widgets/README.md) — which app widgets are still service-coupled, and why.
- [`packages/quark_widgets/README.md`](packages/quark_widgets/README.md) — the widget API reference.
- `packages/quark_widgets/skills/` — task guides for decoupling a page and for writing a widget test.

## Working in this repository

### Use Makefile targets (always)

Use the existing targets to build, run, test, and check the codebase rather than crafting your own shell
commands. If an action needs to be templatized for general usage, add a target rather than running raw
commands. `make help` lists everything; these are the ones that matter day to day.

| Target                          | What it does                                                                |
| ------------------------------- | --------------------------------------------------------------------------- |
| `make setup`                    | gotools, golangci-lint, probe, air, sqlc, swag, flutter, skills, git hooks   |
| `make check`                    | the whole gate: `check/backend`, `check/frontend`, `check/spelling`          |
| `make fix`                      | `dart format`, `go mod tidy`, `gofmt -s -w`                                  |
| `make generate`                 | sqlc and swagger, plus frontend icons, SBOM, and widget docs                 |
| `make test`                     | unit tests only — **not** the API integration tests                          |
| `make test/unit/backend`        | Go unit tests: everything except `internal/server/api/v0/…`                  |
| `make test/integration/backend` | the API handler tests (real filesystem, real gin engine)                     |
| `make test/unit/frontend`       | root `flutter test`, then `make -C packages/<pkg> test/unit` for each package |
| `make watch/backend`            | backend with hot reload, plain HTTP on `:8080`                               |
| `make watch/backend/secure`     | backend with hot reload, HTTPS on `:443`, self-signed                        |
| `make serve/frontend`           | Flutter web dev server                                                       |

GNU make is required. On macOS the system `make` is BSD make and cannot read this Makefile — use `gmake`, which
is what `git/hooks/pre-commit` does.

A new target follows the existing naming: `serve/...` to run something, `build/...`, `check/...`, `test/...`,
and so on. Read the neighboring targets before naming one — `run/docker` next to `serve/frontend` is the kind of
mismatch review sends back.

### What the checks enforce

`make check` is installed as a pre-commit hook (`make setup/hooks`), and `.github/workflows/check.yml` runs the
same targets, so nothing here is advisory.

- **Formatting and lint** — `gofmt`, golangci-lint, `scripts/check-go-structure.bash`, `sqlc vet`,
  `dart format --set-exit-if-changed`, and `flutter analyze`.
- **Generated code is committed.** CI runs `make generate/backend` and `make generate/frontend` and then fails
  on `git diff --exit-code`. The same goes for `make tidy/go` and `make tidy/flutter`: an untidy `go.mod` or
  workspace resolution is a red build.
- **Spelling** — `make check/spelling` runs cspell against `.vscode/cspell.json`, which carries an allowlist of

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [autobutler-org/quark](https://github.com/autobutler-org/quark) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-18 -->
