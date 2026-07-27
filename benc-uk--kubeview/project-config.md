---
trigger: always_on
description: KubeView is a Kubernetes cluster visualization tool providing a real-time graphical view of cluster resources and their relationships. It consists of a **Go backend** serving a REST API and static files, and a **vanilla JavaScript frontend** using ES modules (no bundler). The frontend uses G6 for graph visualization, Alpine.js for UI interactivity, and Bulma for CSS styling.
---

# KubeView - Copilot Instructions

## Project Overview

KubeView is a Kubernetes cluster visualization tool providing a real-time graphical view of cluster resources and their relationships. It consists of a **Go backend** serving a REST API and static files, and a **vanilla JavaScript frontend** using ES modules (no bundler). The frontend uses G6 for graph visualization, Alpine.js for UI interactivity, and Bulma for CSS styling.

### Key Architecture Concepts

- The Go backend connects to Kubernetes via `client-go`, using both dynamic and typed clients.
- Real-time updates flow from Kubernetes informers → SSE broker → frontend `EventSource`.
- Clients are identified by a `clientID` (generated in the browser, stored in `localStorage`) and grouped by namespace for targeted SSE broadcasts.
- The frontend is embedded into the Go binary at build time using `//go:embed` (see `frontend-fs.go` at the project root).
- Kubernetes resources are handled as `unstructured.Unstructured` throughout, keeping the backend generic.
- Secret and ConfigMap data values are redacted to `*REDACTED*` before being sent to the frontend.

### Project Structure

- `server/` — Go backend: API handlers (`api.go`), HTTP routing (`routes.go`), SSE streaming (`sse.go`), configuration (`config.go`), entry point (`main.go`).
- `server/services/` — Kubernetes service layer: cluster connectivity, resource fetching, informer setup, pod logs.
- `frontend/` — Static web app: HTML, CSS, JS (ES modules), images, and third-party libraries in `ext/`.
- `frontend/js/` — Application JavaScript modules.
- `frontend/fragments/` — HTML fragments loaded dynamically at runtime via `loader.js`.
- `frontend-fs.go` — Root-level file that embeds the `frontend/` directory. Must remain at root for `//go:embed` to work.
- `deploy/` — Dockerfile (multi-stage build) and Helm chart for Kubernetes deployment.
- `.dev/` — Development tooling config: ESLint, Prettier, golangci-lint, Air (live reload), and a separate `tools.mod` for Go dev tools.
- `scripts/` — Install script for downloading pre-built binaries.

### API Routes

- `GET /api/namespaces` — List namespaces (also returns cluster metadata).
- `GET /api/fetch/{namespace}?clientID={clientID}` — Fetch all resources in a namespace; also registers the client to receive SSE updates for that namespace.
- `GET /api/logs/{namespace}/{podname}` — Fetch pod logs.
- `GET /api/status` — Server status, version, and build info.
- `GET /updates?clientID={clientID}` — SSE event stream for real-time resource updates.
- `GET /health` — Health check endpoint.
- `GET /` — Serves the main `index.html`.
- `GET /public/*` — Serves embedded static frontend files.

### Build & Dev Commands

- `make run` — Run locally with live reload (uses Air).
- `make build` — Build the Go binary.
- `make lint` — Lint both Go and JS code (CI mode).
- `make lint-fix` — Lint and auto-fix formatting.
- `make test` / `make test-unit` — Run unit tests.
- `make test-integration` — Run integration tests (requires a Kubernetes cluster).
- `make test-coverage` — Run tests with coverage report.
- `make image` — Build container image.
- `make clean` — Remove build artifacts.

---

## Go Coding Instructions

### Style & Conventions

- Use `:=` for variable declarations unless the variable is already declared.
- Use `log.Printf()` and `log.Println()` for logging, never `fmt.Printf()` or `fmt.Println()`.
- Use descriptive emoji prefixes in log messages: `🚀` startup, `💥` errors, `✅` success, `⚓` cluster operations, `🔍` data fetching.
- Organize imports in three groups separated by blank lines: standard library, third-party packages, local packages.
- Use comprehensive file-level banner comments (`// ====...====`) describing each file's purpose.
- Use meaningful short variable names: `gvr` for GroupVersionResource, `ns` for namespace, `k` for Kubernetes service receiver.
- Avoid global variables; pass dependencies as function parameters or struct fields.
- Use `make()` with capacity hints when the approximate size of slices/maps is known.
- Max line length is 120 characters (enforced by `revive` linter).
- Max cyclomatic complexity is 15 (enforced by `cyclop` linter).
- JSON/YAML struct tags must use `goCamel` naming (enforced by `tagliatelle` linter).

### Architecture Patterns

- **Composition over inheritance**: Embed structs/interfaces (e.g., `KubeviewAPI` embeds `*api.Base` from `go-rest-api`).
- **Constructor pattern**: Use `NewServiceName()` functions that return initialized structs.
- **Guard clauses**: Structure HTTP handlers to validate required parameters (namespace, clientID, etc.) and return early on errors.
- **Error responses**: Use `problem.Wrap(statusCode, URI, title, err).Send(w)` from the `go-rest-api` package for RFC 7807 structured error responses. Never use generic `http.Error()`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [benc-uk/kubeview](https://github.com/benc-uk/kubeview) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
