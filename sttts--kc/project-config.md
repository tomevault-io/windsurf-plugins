---
trigger: always_on
description: - Only use Bubble Tea v2: import `charm.land/bubbletea/v2` everywhere (including tests). Do NOT import `charm.land/bubbletea` without `/v2`.
---

# Repository Guidelines

## Dependencies Policy (Important)
- Only use Bubble Tea v2: import `charm.land/bubbletea/v2` everywhere (including tests). Do NOT import `charm.land/bubbletea` without `/v2`.
- Keep module paths consistent with Go’s major version semantics. If a module ships a v2+, the import path must include the `/vN` suffix.
- If you see mixed v0/v1 vs v2 imports, fix them immediately and run `go mod tidy`.
- Example:
  - Correct: `tea "charm.land/bubbletea/v2"`
  - Incorrect: `tea "charm.land/bubbletea"` (will break types between v1/v2)

## Architecture & Project Structure

The application is a **Kubernetes TUI** built with **Go 1.24** and **Bubble Tea v2**, structured around a hierarchical navigation model where resources are treated as "folders" and "files".

### Core Components

1.  **Entry Point (`cmd/kc`)**:
    -   Uses `kong` for CLI argument parsing.
    -   Sets up `controller-runtime` logging (zap).
    -   Initializes the UI via `ui.Run`.

2.  **UI Core (`internal/ui`)**:
    -   **App Model**: The central Bubble Tea model (`App` struct in `app.go`). It manages global state: two panels (`leftPanel`, `rightPanel`), an integrated terminal, and modal overlays.
    -   **Panels**: Each `Panel` is a self-contained component that displays either a list of items (a "Folder") or a content viewer (YAML, logs).
    -   **Navigator**: (`internal/navigation`) Manages the navigation stack for each panel. It treats navigation as a stack of `Folder` objects, supporting "enter" (push) and "back" (pop) operations.

3.  **Data Layer (`internal/tablecache`, `internal/cluster`)**:
    -   **Live Data**: Uses **Informers** (via `controller-runtime`) to maintain a live cache of resources.
    -   **Table Cache**: Decorates standard K8s clients to produce "Table" views (similar to `kubectl get`). It watches for changes and updates the UI in real-time.
    -   **Cluster Pool**: Manages connections to Kubernetes clusters.

4.  **Abstraction (`internal/models`)**:
    -   **Folder Interface**: The core abstraction. A `Folder` provides a list of `Item`s.
    -   **Enterable Interface**: Items that can be "entered" (like a Namespace or a Pod) implement this interface to return a new `Folder`.
    -   **Universal Navigation**: Allows uniform navigation through Contexts → Namespaces → Resource Groups → Resources → Objects → Sub-resources.

### Key Characteristics
-   **Server-Side Rendering**: Relies on server-side printing (Table output) from Kubernetes to determine how to display resources, ensuring compatibility with CRDs.
-   **"Vibe-Coded"**: Embraces an AI-assisted, experimental coding style, prioritizing features and "feel" over strict traditional rigor, while maintaining a solid modular architecture.

### Directory Layout
-   `cmd/kc/`: Application entrypoint.
-   `internal/ui/`: TUI components (App, Panel, Terminal).
-   `internal/models/`: Core interfaces (Folder, Item, Enterable).
-   `internal/tablecache/`: Live data caching and Table view materialization.
-   `internal/cluster/`: Kubeconfig and client pool management.
-   `internal/navigation/`: Stack-based navigation logic.
-   `pkg/`: Reusable helpers (config, etc.).
-   `examples/`: Minimal runnable samples.

## Build, Test, and Development Commands
- Build binary: `go build -o kc ./cmd/kc`
- Run binary: `./kc`
- Run without building: `go run ./cmd/kc`
- Run examples: `go run examples/handler/main.go`
- Headless TUI driver: `go run ./cmd/bubbleheadless -- go run ./cmd/kc`
- All tests (verbose): `go test ./... -v`
- With coverage: `go test ./... -cover`
- Static checks: `go vet ./...`
- Tidy modules (after dep changes): `go mod tidy`

### Headless Bubbleterm Wrapper
- `cmd/bubbleheadless` is a REPL-style driver around bubbleterm for end-to-end automation in non-interactive environments.
- Launch it with `go run ./cmd/bubbleheadless -- <app command...>`; e.g. `go run ./cmd/bubbleheadless -- go run ./cmd/kc`.
- Once started, send commands over stdin:
  - `key <token>` / `text <string>` to feed keystrokes,
  - `mouse <button> <x> <y> <press|release|motion>` for mouse events,
  - `screen [ansi|plain]` to dump the current framebuffer,
  - `snapshot <file.png>` to write a PNG render of the terminal,
  - `resize <cols> <rows>` and `wait` / `exit` for session control.
- Designed for CI/e2e test harnesses—script it from shell pipelines or expect-like drivers without allocating a PTY.

## Coding Style & Naming Conventions
- Format: `go fmt ./...` (CI expects gofmt-clean code).
- Lint mindset: prefer small packages, clear interfaces, early returns.
- Naming: package names lower-case, no underscores; exported identifiers `CamelCase`, unexported `camelCase`.
- Errors: wrap with `%w` (e.g., `fmt.Errorf("reading config: %w", err)`).
- Files: group closely related types; avoid large god files.

## Logging Guidelines
- Always import controller-runtime logging as `ctrllog` (e.g., `ctrllog "sigs.k8s.io/controller-runtime/pkg/log"`).
- Thread a `context.Context` through the call chain and retrieve loggers with `ctrllog.FromContext(ctx)`; never grab the global logger directly.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sttts/kc](https://github.com/sttts/kc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
