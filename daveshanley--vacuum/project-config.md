---
trigger: always_on
description: Operational guide for AI agents working in `github.com/daveshanley/vacuum`.
---

# AGENTS.md

Operational guide for AI agents working in `github.com/daveshanley/vacuum`.

## TL;DR

- vacuum is a Go CLI and library for linting OpenAPI, AsyncAPI and JSON Schema documents, generating reports, bundling OpenAPI and JSON Schema documents where supported, running an LSP server, and generating OpenAPI docs through printing press.
- Main entry point: `vacuum.go` -> `cmd.Execute(...)` -> `cmd.GetRootCommand()`.
- The default OpenAPI and AsyncAPI user path is `vacuum lint <api-description>`. JSON Schema linting uses `vacuum schema <schema>` or `vacuum schema lint <schema>`.
- OpenAPI and AsyncAPI lint/report commands share helpers in `cmd/build_results.go`, `cmd/lint_shared.go`, and `motor/`. AsyncAPI context and default-ruleset wiring lives in `asyncapi/`, `cmd/asyncapi_ruleset.go`, `motor/asyncapi_applicator.go`, and `rulesets/asyncapi_rules.go`. JSON Schema command wiring lives in `cmd/schema*.go` and still executes rules through `motor/`.
- Go version is `1.25.0`. Node is only needed for the HTML report UI and npm package wrapper.
- The interactive HTML report is compiled only when UI assets exist and builds use `-tags html_report_ui`.
- Release/CI-shaped Go checks should usually run with `GOWORK=off` so local sibling checkouts do not hide committed module problems.
- Keep changes narrow. Do not rewrite generated assets, dependency locks, or docs unless the task requires it.

## Verify Changes

Run the smallest useful check first, then broaden when touching shared behavior.

```bash
gofmt -w <changed-go-files>
go test ./...
```

For official-build behavior, HTML report changes, or anything touching `html-report/`:

```bash
./scripts/build-ui-assets.sh
go test -tags html_report_ui ./...
make build
```

For release-shaped dependency, install, or CI validation:

```bash
GOWORK=off GOCACHE=/tmp/go-build go test ./...
GOWORK=off GOCACHE=/tmp/go-build go build ./...
```

For `html-report/ui` package-only changes:

```bash
cd html-report/ui
npm ci
npm run build
npm run lint
```

Before handoff for docs-only or config-only edits:

```bash
git diff --check -- <changed-files>
```

For JSON Schema command, ruleset, or schema Doctor changes, prefer focused checks before broader package runs:

```bash
go test ./cmd -run 'Schema'
go test ./motor -run 'JSONSchema'
go test ./jsonschema ./functions/jsonschema ./functions/schemachecks ./rulesets
```

For AsyncAPI linting, ruleset, or context changes, prefer focused checks before broader package runs:

```bash
go test ./cmd -run 'AsyncAPI|Lint|Report|Dashboard|GenerateRuleset'
go test ./motor -run 'AsyncAPI'
go test ./asyncapi ./functions/asyncapi ./rulesets
```

## Repo Map

```text
vacuum.go                    binary entry point and ldflags pass-through
cmd/                         Cobra commands, CLI flags, rendering, reports, docs and schema commands
asyncapi/                    AsyncAPI context, detection, and libasyncapi bridge helpers
motor/                       rule execution engine, document/index setup, result collection
model/                       rules, result models, reports, categories, test fixtures
rulesets/                    built-in rulesets, schemas, rule aliases, example rulesets
functions/core/              Spectral-compatible core rule functions
functions/asyncapi/          AsyncAPI-specific rule functions
functions/jsonschema/        JSON Schema rule functions and synthetic validation rules
functions/openapi/           OpenAPI-specific rule functions
functions/owasp/             OWASP rule functions
functions/schemachecks/      Shared schema sanity/type checks used by OpenAPI and JSON Schema rules
jsonschema/                  JSON Schema dialect, metaschema, Doctor, and reference helpers
plugin/javascript/           JavaScript custom-function runtime, event loop, fetch support
plugin/sample/               sample Go and JS custom functions
language-server/             OpenAPI LSP server integration
html-report/                 Go HTML report generator and embedded asset switch
html-report/ui/              TypeScript/Webpack UI bundle for HTML reports
tui/                         terminal UI, dashboard, tables, styles, markdown rendering
utils/                       change detection, ignore matching, path location, HTTP helpers
parser/                      JSON/YAML schema validation helpers
vacuum-report/               persisted report format and JUnit support
upgrade/                     release lookup, update notices, install-method detection, self-upgrade actions
npm-install/                 npm postinstall binary downloader for @quobix/vacuum
scripts/build-ui-assets.sh   builds HTML report UI assets for official builds
BUILD_PACKAGING.md           package-manager build and ldflags guidance
```

## Command Surfaces

Root command registration lives in `cmd/root.go`. Current subcommands include:

- `lint`
- `report`
- `spectral-report`
- `html-report`
- `dashboard`
- `docs`
- `generate-ruleset`
- `generate-ignorefile`
- `version`
- `language-server`
- `upgrade`
- `bundle`
- `schema`
- `apply-overlay`
- `open-collection`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [daveshanley/vacuum](https://github.com/daveshanley/vacuum) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
