---
trigger: always_on
description: Rill is a business intelligence platform built around the following principles:
---

## What is Rill

Rill is a business intelligence platform built around the following principles:

- Code-first: configure projects using versioned and reproducible source code in the form of YAML and SQL files.
- Full stack: go from raw data sources to user-friendly dashboards powered by clean data with a single tool.
- Declarative: describe your business logic and Rill automatically runs the infrastructure, migrations and services necessary to make it real.
- OLAP databases: you can easily provision a fast analytical database and load data into it to build dashboards that stay interactive at scale.

## Architecture

Users define projects as YAML and SQL files that describe _resources_ — connectors, models, metrics views, dashboards, and more — organized in a DAG. The runtime **parses** project files into resources and **reconciles** each resource to its desired state (e.g., materializing a model into DuckDB, validating a metrics view's dimensions and measures). On the frontend, metrics views power two dashboard types: **explore dashboards** (drill-down, slice-and-dice) and **canvas dashboards** (free-form charts and tables). The platform also supports alerts, scheduled reports, custom APIs, and a built-in AI assistant.

Two deployment modes share the same codebase:

- **Rill Developer** — local application for data engineers. A single Go binary that embeds the CLI, runtime, and `web-local` frontend. Code-first, version-controlled workflow.
- **Rill Cloud** — hosted platform for teams. Runs the `admin` service, runtime(s), and `web-admin` frontend as separate services. Adds auth, billing, multi-tenancy, and collaboration.

### Key Directories

- `runtime/` — data plane: orchestration, queries, connectors, access policies, reconcilers
- `admin/` — cloud control plane: auth, billing, provisioning, project management
- `cli/` — CLI and local application server
- `web-common/` — shared frontend library consumed by both `web-local` and `web-admin`
- `web-local/` — local frontend (Rill Developer)
- `web-admin/` — cloud frontend (Rill Cloud)
- `proto/` — gRPC/protobuf API definitions (source of truth for all APIs)

## Development

### Common Commands

- **Build CLI**: `make cli` (Go binary + embedded frontend)
- **Build CLI only**: `make cli-only` (skip frontend, faster)
- **Local dev**: `rill devtool start local`
- **Cloud dev**: `rill devtool start cloud`
- **Test Go**: `go test ./...`
- **Test frontend (unit, web-common)**: `npm run test -w web-common` (fast, use for tight feedback loops)
- **Test frontend (unit, web-admin)**: `cd web-admin && npx vitest run src/path/to/spec.ts` (must run from `web-admin/` so vitest picks up the `@rilldata/web-admin` alias)
- **Test frontend (e2e)**: `npm run test -w web-local` or `npm run test -w web-admin` (Playwright, slow)
- **Lint/format frontend**: `npm run quality`
- **Regenerate docs**: `make docs.generate` (run after changes to `proto/`, `cli/` or `runtime/parser`)
- **Regenerate API bindings:** `make proto.generate` (run after changes to `proto/`)

### Adding or Changing APIs

APIs are defined in `.proto` files and mapped to REST via gRPC-Gateway. See `proto/README.md` for conventions.

1. Define endpoint in the relevant `.proto` file under `proto/rill/`
2. Run `make proto.generate`
3. Implement handler in `runtime/server/` (or `admin/server/`)

See `runtime/README.md` for details.

Frontend API clients are auto-generated from proto definitions using **Orval**. Do not hand-edit files under `web-common/src/runtime-client/` — regenerate them instead.

## Code Conventions

### Go

General rules for writing Go code:

- Write Go code in the style of Uber's Go style guide.
- Use `golangci-lint` for linting. After making Go changes, run `golangci-lint run ./path/to/package/` on the affected packages to catch issues before committing.
- Non-trivial directories should have a `README.md`. If a directory has a README, always read it before making changes in that directory.
- Functions should be sorted roughly in call order; functions should be grouped by receiver; plain utility functions belong towards the end of a file.
- When adding a field to a struct or interface, don't automatically put at it at the end of the field list, instead put it where it makes the most sense (i.e. grouped with related fields and higher up than less important fields).
- Prefer colons or semi-colons in code comments instead of hyphens or dashes. This keeps comments shorter, which makes them more readable in a monospace font.
- Use semantic line breaks in comments: break at the end of a sentence, or after a comma, semicolon, or colon in longer sentences, rather than wrapping at a fixed column. Do not break a line mid-phrase to hit a character limit; a long line that expresses one complete thought reads better on a wide screen than an arbitrary mid-phrase wrap.
- Before adding a dependency, check for newer major versions. Major versions 2+ require the `/vN` suffix in the import path (e.g., `go get github.com/foo/bar/v3@latest`). Without the suffix, `go get` only fetches v1.x.
- Avoid short utility functions that are only used once; it is usually more readable to inline these in their parent function.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rilldata/rill](https://github.com/rilldata/rill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
