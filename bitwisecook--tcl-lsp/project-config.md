---
trigger: always_on
description: handles the full Tcl specification; the compiler only inlines what it can
---

# AGENTS.md — development guide for AI agents

## Project overview

tcl-lsp is a Tcl Language Server Protocol implementation written in Python
(server) with editor integrations in TypeScript (VS Code), Rust (Zed), and
Gradle/Kotlin (JetBrains). It supports Tcl 8.4–9.0, F5 iRules/iApps, and EDA
tool dialects.

## Repository layout

```
lsp/             Python LSP server runtime and feature wiring
core/            Reusable Tcl parser/compiler/analysis modules
vm/              Bytecode VM, interpreter, and REPL
debugger/        Interactive Tcl debugger (CLI, VM/tclsh/tkinter backends)
editors/vscode/  VS Code extension (TypeScript)
editors/         Other editor integrations (Neovim, Zed, Emacs, Helix, Sublime, JetBrains)
explorer/        Web-based compiler explorer (Pyodide GUI)
tests/           Python test suite (pytest)
scripts/         Build and release automation
ai/              AI integrations (Claude skills, MCP server)
samples/         Sample Tcl and iRules code
```

## Prerequisites

- Python 3.10+ with [uv](https://docs.astral.sh/uv/)
- Node.js 20+ with npm

### Claude Code on the web — pre-installed toolchains and sources

The SessionStart hook in [`.claude/hooks/session-start.sh`](.claude/hooks/session-start.sh)
prepares remote sessions (containers where `CLAUDE_CODE_REMOTE=true`) with
the language toolchains and Tcl source trees the repo needs. It runs on
local machines as a no-op, so laptops are never touched. Everything listed
here is ready before Claude starts taking instructions — **no manual
`apt install` or curl step is required**.

| Tool / source    | Version       | Install path                    | On `PATH` as              |
|------------------|---------------|---------------------------------|---------------------------|
| rsync, xz-utils  | distro        | `/usr/bin/`                     | `rsync`, `xz`             |
| Zig              | 0.16.0        | `/opt/zig-0.16.0/`              | `/usr/local/bin/zig`      |
| Wasmtime         | v43.0.1       | `/opt/wasmtime-43.0.1/`         | `/usr/local/bin/wasmtime` |
| rustup + Rust    | stable 1.95.0 | `/root/.rustup`, `/root/.cargo` | `/usr/local/bin/{cargo,rustc,rustup,rustfmt,clippy-driver}` |
| Tcl 8.4 source   | 8.4.20        | `tmp/tcl8.4.20/`                | —                         |
| Tcl 8.5 source   | 8.5.19        | `tmp/tcl8.5.19/`                | —                         |
| Tcl 8.6 source   | 8.6.16        | `tmp/tcl8.6.16/`                | —                         |
| Tcl 9.0 source   | 9.0.3         | `tmp/tcl9.0.3/`                 | —                         |
| tcllib           | 2.0           | `tmp/tcllib-2.0/`               | —                         |

Notes on the fetched sources:

- Tcl and tcllib are full source trees (`generic/`, `unix/`, `win/`, `tests/`,
  `library/`, `doc/`, …) pulled as release tarballs from
  `codeload.github.com`. Tarballs are GitHub-CDN cached, smaller than a git
  clone, and friendlier to the upstream Tcl project than hitting
  `tcl.tk`/`sourceforge.net` on every cold session.
- Zig is fetched via the community mirror pool listed at
  [`community-mirrors.txt`](https://ziglang.org/download/community-mirrors.txt);
  the hook shuffles the pool, falls back to `ziglang.org` as the last resort,
  and verifies the x86_64-linux tarball against the published SHA-256.
- The hook is idempotent — warm containers re-run it and finish in seconds.

To bump any of these versions, edit the pinned variables at the top of
[`.claude/hooks/session-start.sh`](.claude/hooks/session-start.sh)
(`ZIG_VERSION`, `WASMTIME_VERSION`, `RUST_VERSION`, `TCLLIB_TAG` /
`TCLLIB_VERSION`) and, for Tcl, the version/tag maps in
[`.claude/skills/fetch-tcl-source/fetch_tcl_source.sh`](.claude/skills/fetch-tcl-source/fetch_tcl_source.sh).
For Zig, refresh `expected_sha` in the hook to match the new x86_64-linux
tarball's SHA-256 from `https://ziglang.org/download/index.json`.

### Version requirements — sources of truth and update checklist

The **source of truth** for each minimum version:

| Requirement | Source of truth              | File                  |
|-------------|------------------------------|-----------------------|
| Python      | `requires-python`            | `pyproject.toml`      |
| Node.js     | CI matrix                    | `.github/workflows/ci.yml` |

When changing a minimum version, update **all** of these locations:

- `pyproject.toml` — `requires-python` and `[tool.ruff]` `target-version`
- `.github/workflows/ci.yml` — `python-version` matrix and `node-version` values
- `Makefile` — Prerequisites comment block at the top
- `AGENTS.md` — Prerequisites section (this file)
- `README.md` — Prerequisites / requirements section
- `editors/vscode/package.json` — `tclLsp.pythonPath` description text
- `editors/jetbrains/README.md` — Python version references
- `editors/neovim/README.md` — Python version in zipapp instructions

## Build system

The project uses GNU Make. Key targets:

| Target             | Purpose                                  |
|--------------------|------------------------------------------|
| `make prep-pr`     | **Fast pre-PR gate** (format + lint + typecheck + fast tests) — run this before every PR |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bitwisecook/tcl-lsp](https://github.com/bitwisecook/tcl-lsp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
