---
trigger: always_on
description: - Use Go 1.27 or later, as declared in `go.mod`, `tests/go.mod`, and `go.work`. The workspace contains two modules: the root module and `tests/`. Root `go test ./...` does not include the E2E module.
---

# Repository Instructions

## Code Boundaries

- Use Go 1.27 or later, as declared in `go.mod`, `tests/go.mod`, and `go.work`. The workspace contains two modules: the root module and `tests/`. Root `go test ./...` does not include the E2E module.
- `container.Plugins()` in `container/plugins.go` defines the default plugin set for the Endure container. See Plugins And Endure below.
- The CLI starts at `cmd/rr/main.go` and `internal/cli/root.go`. Lifecycle changes can affect three separate implementations: `internal/cli/serve/command.go` (non-Windows), `internal/cli/serve/command_windows.go` (Windows), and `lib/roadrunner.go` (embedding API).

## Plugins And Endure

- Every bundled plugin is a separate Go module and GitHub repository in the `roadrunner-server` organization with the import path `github.com/roadrunner-server/<name>/v6`. The Temporal plugin is `github.com/temporalio/roadrunner-temporal/v6`. Root `go.mod` pins every bundled plugin. `tests/go.mod` pins only the plugins that the E2E tests import.
- Organization repositories: plugins at `https://github.com/roadrunner-server/<name>`, Endure at `https://github.com/roadrunner-server/endure`, error kinds at `https://github.com/roadrunner-server/errors`, cross-plugin contracts at `https://github.com/roadrunner-server/api-plugins`, generated protobuf Go bindings at `https://github.com/roadrunner-server/api-go`, protobuf sources at `https://github.com/roadrunner-server/api` (not a Go module), the worker pool at `https://github.com/roadrunner-server/pool`, example plugins at `https://github.com/roadrunner-server/samples`, the build tool at `https://github.com/roadrunner-server/velox`, and the user documentation at `https://github.com/roadrunner-server/docs`.
- The build compiles plugins from the module cache. `go.work` covers both modules, so Go resolves each plugin to the highest version that `go.mod` or `tests/go.mod` requires. `go.mod` has no `replace` directives. To read a plugin's source at the pinned version, run `go list -m -f '{{.Dir}}' github.com/roadrunner-server/<name>/v6`. The `master` branch on GitHub can be ahead of that version.
- To build or test against a local plugin checkout, write a `go.work` file outside this repository with absolute `use` paths for the repository root, `tests`, and the plugin directory. Run Go commands with `GOWORK=<absolute path to that file>`. Never commit a `replace` directive or a `use` entry that points outside this repository.
- Run a plugin's unit tests with `go test ./...` from the plugin repository root. When the plugin has a `tests/` module, run its E2E tests from that directory, as the plugin CI does. That `tests/go.mod` replaces the plugin module with the parent directory.
- Endure (`github.com/roadrunner-server/endure/v2`) is the dependency injection container. Its source is at `https://github.com/roadrunner-server/endure/blob/master/`: `container.go` declares the plugin interfaces, `edges.go` validates `Init` and builds the graph, `init.go` calls `Init`, handles disabled plugins, and registers `Provides` values, and `collects.go` runs the `Collects` callbacks. Read them at the pinned version before you change a plugin's `Init` parameters, `Provides`, or `Collects`.
- Endure resolves dependencies by reflection over the `Init` method parameters. Every registered struct needs an `Init` method. Each `Init` parameter must be an interface type, and `Init` must return exactly one `error` value. A missing `Init`, a struct parameter, or a primitive parameter stops the whole container.
- These interfaces are optional: `Service` (`Serve() chan error`, `Stop(context.Context) error`), `Named` (`Name() string`), `Provider` (`Provides() []*dep.Out`), `Collector` (`Collects() []*dep.In`), and `Weighted` (`Weight() uint`). Build `dep.Out` with `dep.Bind` and `dep.In` with `dep.Fits`. Both accept only a pointer to an interface type, such as `(*Middleware)(nil)`, and panic on other types. A method named in `Provides` takes no arguments and returns exactly one value.
- Endure calls `Stop` of every active `Service` plugin concurrently, each with its own context that expires after `grace_period` (`stop.go` in the Endure repository). `grace_period` is a per-plugin timeout, not an overall deadline.
- When two plugins satisfy the same `Init` parameter, Endure sorts the candidates by `Weight()` in descending order and takes the first. `tests/mock/logger.go` provides the `Logger` interface with weight 100 to replace the real logger plugin in E2E tests.
- A plugin declares the interfaces it consumes in its own package, usually near the top of `plugin.go`. `http` and `grpc` declare most of them in `api/interfaces.go`. Shared contracts live in `github.com/roadrunner-server/api-plugins/v6`, and plugins import its `jobs`, `kv`, and `status` packages. A plugin that needs a logger declares its own interface with `NamedLogger(name string) *slog.Logger`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [roadrunner-server/roadrunner](https://github.com/roadrunner-server/roadrunner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
