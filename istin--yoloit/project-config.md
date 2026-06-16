---
trigger: always_on
description: This guide outlines non-obvious developer commands, compilation pipelines, strict coding standards, and agent operational gotchas for this codebase.
---

# YoLoIT Agent Guidance (`AGENTS.md`)

This guide outlines non-obvious developer commands, compilation pipelines, strict coding standards, and agent operational gotchas for this codebase.

---

## ⚡ Non-Obvious Commands & Local Workflows

- **Interactive macOS Launch & Hot-Reload**:
  - Run: `./run.sh` (starts Flutter with a FIFO pipe at `/tmp/yoloit_flutter_stdin`).
  - Hot Reload: `./hot_reload.sh` (triggers hot-reload on the running instance by writing `r` to the pipe).
- **Dual Target & Web Client Compilation**:
  - Desktop app uses `lib/main.dart`.
  - Collaboration web client uses `lib/main_web.dart` and must be built via:
    `flutter build web --release --target lib/main_web.dart`
- **Windows Build Patch**:
  - Windows builds have a known Cargokit symlink resolution bug. **Always** run this patch before compiling on Windows:
    `.\windows\flutter\tools\patch_cargokit.ps1`
- **Git Submodules**:
  - Submodules are used for `packages/mermaid_renderer_flutter` and `third_party/flutter_local_models`. Initialize them using:
    `git submodule update --init --recursive --depth=1`

---

## 🛠️ YoLoIT CLI & App Development Workflows

- **CLI-First Development Philosophy**:
  - All features, actions, and state mutations must be designed **CLI-first**. Everything in the application—including managing and interacting with widgets on the boards—must be fully controllable via the `yoloit` CLI.
  - New functionality must expose commands that map seamlessly to both the terminal CLI and LLM Tools.
  - Every command must include a clear, concise, and **human-friendly description** so both humans and agents can easily discover, understand, and use them.
- **Long-Running Dev Processes**:
  - Never run long-running servers or builders directly in the chat foreground. Use the `yoloit` CLI:
    `yoloit panel:create "<board>" board.run "Run"` to create a runner panel, then:
    `yoloit do "<board>" "<panel>" run '{"id":"..."}'` to start the process persistently.
- **Multi-Step Board Mutations**:
  - Prefer `yoloit board:apply` with a YAML specification instead of sending many imperative single commands.
- **Custom JS App (Widget) Development**:
  - Always run `yoloit app:dev-skill` first to read the JavaScript API and UI rules.
  - **No Local Install Needed**: Develop `widget.js` and `manifest.json` in the current working directory, and control/test them directly:
    `yoloit app:run .` (open) | `yoloit app:reload .` (hot-reload) | `yoloit app:logs .` (stream console)

---

## 🧭 CodeGraph Usage: CLI Only

- **Do not rely on the CodeGraph MCP server for this repository.**
  - The repo is large enough that the CodeGraph file watcher can hit macOS file descriptor limits (`EMFILE: too many open files, watch`) and make MCP startup time out.
  - Prefer explicit `codegraph` CLI calls from the terminal. They are deterministic, easy to retry, and do not block Codex startup.
- **Use CodeGraph CLI for structural code questions before falling back to text search:**
  - Symbol or API context: `codegraph context "task or area"`
  - Symbol lookup: `codegraph query "SymbolName"`
  - Callers: `codegraph callers "SymbolName"`
  - Callees: `codegraph callees "SymbolName"`
  - Impact analysis: `codegraph impact "SymbolName"`
  - File list from index: `codegraph files`
  - Index health: `codegraph status --json`
- **Keep the index fresh manually:**
  - Run `codegraph sync` after meaningful edits before asking structural questions.
  - If the index looks stale or locked, run `codegraph status` first and follow its output; do not start or debug the MCP server.
- **When using CLI output in answers or implementation work:**
  - Treat CodeGraph CLI results as the structural source of truth for definitions, call relationships, and impact.
  - Use `rg` only for literal text, comments, log messages, config keys, and after CodeGraph has already identified specific files.

---

## 📦 Repomix Code Snapshots

A pre-configured `repomix.config.json` lives at the repo root. It excludes `third_party`, `packages` (submodules), generated Dart files (`.g.dart`, `.freezed.dart`, `.mocks.dart`), and training data (`assets/command_catalog/**/*.jsonl`).

- **Install** (one-time): `npm install -g repomix`
- **Full project snapshot**: `repomix .`
  - Output: `snapshots/repomix-output.xml` (~500K tokens, ~2 MB, ~1.5s)
- **Dart-only snapshot**:
  ```bash
  find lib test -name '*.dart' ! -name '*.g.dart' ! -name '*.freezed.dart' ! -name '*.mocks.dart' | repomix --stdin --compress --output snapshots/dart-only.xml
  ```
- **Single / multiple specific files** (e.g., after `rg`/`codegraph` search):
  ```bash
  # One file
  echo 'lib/features/board/ui/board_view.dart' | repomix --stdin --compress --output snapshots/target.xml

  # Several files
  printf 'lib/app.dart\nlib/main.dart\n' | repomix --stdin --compress --output snapshots/target.xml
  ```
  > ⚠️ Multiple `--include` flags do **not** work reliably — always use `--stdin` for specific file lists.
- **Tips**:
  - `--compress` uses Tree-sitter to keep signatures and replace method bodies with `⋮----`. It captures **all** methods (public, private, protected).
  - `snapshots/` is `.gitignore`d — never commit generated XML files.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [IstiN/yoloit](https://github.com/IstiN/yoloit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
