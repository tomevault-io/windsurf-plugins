---
trigger: always_on
description: handles something, consult its source.
---

# pi-coding-agent — Development Guide

Emacs frontend for the [pi coding agent](https://pi.dev).
Two-window UI: markdown chat buffer + prompt composition buffer.
Communicates with the pi CLI via JSON-over-stdio (RPC).

## Module Architecture

Eight source modules with a strict dependency chain (no cycles):

```
pi-coding-agent.el              ← entry point, autoloads
  ├── pi-coding-agent-menu.el   ← transient menu, session management
  ├── pi-coding-agent-input.el  ← input buffer, history, completion
  └── pi-coding-agent-render.el ← chat rendering, tool output
        ├── pi-coding-agent-table.el  ← display-only table decoration
        └── pi-coding-agent-ui.el ← shared state, faces, modes
              ├── pi-coding-agent-core.el ← JSON/RPC protocol
              └── pi-coding-agent-grammars.el ← tree-sitter grammar recipes
```

External package dependency:

- `md-ts-mode` ← tree-sitter markdown major mode used by chat buffers
  (loading `pi-coding-agent` must not globally claim unrelated Markdown files)

`menu.el` and `input.el` are siblings — neither requires the other.
They communicate through shared variables in `ui.el` (e.g., `--commands`).

`table.el` and `ui.el` are siblings under `render.el`.  `table.el`
depends on `ui.el` for visible-text extraction and scroll preservation.

Cross-module state mutations use accessor functions defined in `ui.el`
(e.g., `--set-process`, `--set-aborted`, `--push-followup`).  Within a
module, direct `setq` is fine.

## Source Files

| File | Purpose |
|------|---------|
| `pi-coding-agent.el` | Entry point, autoloads, `--setup-session` |
| `pi-coding-agent-core.el` | JSON parsing, line buffering, RPC protocol |
| `pi-coding-agent-ui.el` | Shared state, faces, customization, modes, display primitives, header-line |
| `pi-coding-agent-render.el` | Streaming chat rendering, tool output, fontification, diffs |
| `pi-coding-agent-table.el` | Display-only pipe table decoration, wrapping, overlay management |
| `pi-coding-agent-input.el` | Input history, isearch, send/abort, file/path/slash completion, queuing |
| `pi-coding-agent-menu.el` | Transient menu, session management, model selection, commands |
| `pi-coding-agent-grammars.el` | Tree-sitter grammar recipes, install prompts, `M-x pi-coding-agent-install-grammars` |

## Test Files

| File | Covers |
|------|--------|
| `test/pi-coding-agent-core-test.el` | Core/RPC protocol |
| `test/pi-coding-agent-ui-test.el` | Buffer naming, modes, session dir, startup header, grammar install |
| `test/pi-coding-agent-render-test.el` | Response display, tools, diffs |
| `test/pi-coding-agent-table-test.el` | Table decoration, overlays, streaming, resize |
| `test/pi-coding-agent-input-test.el` | History, send/abort, queuing, completion |
| `test/pi-coding-agent-menu-test.el` | Session management, transient menu, reconnect |
| `test/pi-coding-agent-build-test.el` | Batch helper scripts for dependency and grammar installation |
| `test/pi-coding-agent-test.el` | Entry point / cross-module integration |
| `test/pi-coding-agent-test-common.el` | Shared fixtures: mock-session macro, toolcall helpers, fake-pi launch helpers |
| `test/pi-coding-agent-integration-test-common.el` | Shared integration backend helpers and contract macros |
| `test/pi-coding-agent-integration-test-common-test.el` | Unit tests for shared integration helper macros |
| `test/pi-coding-agent-integration-rpc-smoke-test.el` | Cheap shared fake/real RPC canaries |
| `test/pi-coding-agent-integration-prompt-contract-test.el` | Shared fake/real prompt lifecycle + abort contracts |
| `test/pi-coding-agent-integration-session-contract-test.el` | Shared fake/real session-file persistence contract |
| `test/pi-coding-agent-integration-steering-contract-test.el` | Shared fake/real steering contract |
| `test/pi-coding-agent-integration-tool-contract-test.el` | Shared fake/real tool execution contract |
| `test/pi-coding-agent-integration-test.el` | Integration suite entry point (loads all shared contract modules) |
| `test/pi-coding-agent-gui-tests.el` | GUI tests (require display or xvfb) |

## Other Files

| File | Purpose |
|------|---------|
| `Makefile` | Build, test, lint targets |
| `bench/pi-coding-agent-bench.el` | Table rendering benchmark harness (xvfb GUI or batch) |
| `bench/run-bench.sh` | Benchmark runner script; `--batch` for headless lane |
| `bench/fixtures/tables.md` | Sample pipe tables used by the benchmark |
| `scripts/check.sh` | Pre-commit hook: byte-compile + lint + tests |
| `scripts/pi-coding-agent-build.el` | Shared batch helpers for dependency and grammar installation |
| `scripts/install-deps.el` | Batch script: install required Emacs package dependencies |
| `scripts/install-ts-grammars.el` | Batch script: install tree-sitter grammars |

## Running Tests

Run all unit tests:
```bash
make test
```

Run tests for a single module:
```bash
make test-core
make test-ui
make test-render
make test-input
make test-menu
make test-build
```

Run shared integration contracts:
```bash
make test-integration          # fake + real
make test-integration-fake     # fake only, fast
make test-integration-real     # real only
```

Run a filtered subset by ERT pattern:
```bash
make test SELECTOR=fontify-buffer-tail
make test SELECTOR=toolcall-delta

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dnouri/pi-coding-agent](https://github.com/dnouri/pi-coding-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
