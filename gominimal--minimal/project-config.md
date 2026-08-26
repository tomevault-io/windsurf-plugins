---
trigger: always_on
description: Orientation for coding agents and contributors working in this repository.
---

# AGENTS.md

Orientation for coding agents and contributors working in this repository.
Tool-neutral and canonical: tool-specific instruction files (e.g.
`CLAUDE.md`) include or link this document rather than restating it.

## Orientation

Minimal is a declarative, content-addressed package/build system plus a
session plane for sandboxed development environments. The system is two
cooperating planes: the **build plane** (declarative packages → dependency
graph → sandboxed builds → content-addressed cache) and the **session plane**
(long-lived isolated dev environments, natively on Linux or inside a libkrun
microVM). Start with [docs/architecture.md](docs/architecture.md).

Four binaries come out of this workspace:

| Binary | Role | Reference |
|---|---|---|
| `min` | Session CLI (built from the `minimal` crate; its `[[bin]]` target is `min`) | [docs/reference/cli-min.md](docs/reference/cli-min.md) |
| `mip` | Package/build CLI | [docs/reference/cli-mip.md](docs/reference/cli-mip.md) |
| `minimald` | Session daemon: SSH server hosting sessions and task/sandbox executions | [docs/reference/cli-minimald.md](docs/reference/cli-minimald.md) |
| `minvmd` | MicroVM host daemon: boots Linux guests via libkrun, bridges host UDS to in-VM vsock | [docs/reference/cli-minvmd.md](docs/reference/cli-minvmd.md) |

The CLI reference overview is [docs/reference/cli.md](docs/reference/cli.md).

## Crate map

31 crates. One line each; the long-form map with plane assignments is in
[docs/architecture.md](docs/architecture.md) §3.

| Crate | Role |
|---|---|
| `args` | Types for the argument schema of tasks and sideload parameters. |
| `async-dialog` | Interactive terminal prompts over any async reader/writer (no TTY required). |
| `check` | `mip check` linting of `minimal.toml`, packages, profiles, and stacks. |
| `checkouts` | Git checkouts of upstream layer repositories at pinned versions. |
| `common` | Common types and utilities (e.g. `SpecHash`) used across the codebase. |
| `decode` | Evaluates a Nickel config layer into in-memory packages/profiles/stacks. |
| `diagnostics` | App-agnostic machinery for diagnostic support bundles. |
| `graph` | In-memory dependency graph; its `planner` module orders builds. |
| `lcache` | Local cache of built artifacts, keyed by `SpecHash`. |
| `mctx` | Top-level 'minimal context' API tying configuration, decoding, graph, and cache together. |
| `mfile` | Finding and reading the `minimal.toml` file. |
| `minimal` | The `min` session CLI, which pairs with and talks to `minimald`. |
| `minimal-client` | SSH client transport to `minimald` over the UDS bridge, shared by `min` and the TUI. |
| `minimal-tui` | `min dash`: the session-manager TUI (ratatui/crossterm, Elm-style loop). |
| `minimald` | The session daemon: an SSH server hosting sessions and task/sandbox executions. |
| `minimald-rpc` | Wire contract for `minimald`'s oneshot SSH RPCs. |
| `minvmd` | Host daemon that boots Linux microVMs via libkrun and bridges host UDS to in-VM vsock. |
| `mip` | The Minimal package/build CLI. |
| `mlog` | JSON file-log layer both `minimald` and `minvmd` write through; one definition of the on-disk log format. |
| `op` | Complex operations over the graph and packages (builds, cache object construction). |
| `orchestrator` | Runtime orchestration of builds behind a pluggable `Backend`. |
| `ot` | Operation tracking for progress rendering (render-agnostic core + drivers). |
| `paths` | Realm-tagged path types distinguishing host, sandbox, and daemon filesystems. |
| `rcache` | Remote cache: fetch/upload build artifacts over the network. |
| `remote-client` | Client for the Remote Execution Service, driving remote builds against the graph. |
| `remote-proto` | Protobuf / wire types for the Remote Execution Service (RES). |
| `sandbox2` | Low-level sandbox implementation (Linux user + mount namespaces). |
| `sessions` | Session primitives: lifecycle hooks, loadouts, and the composition pipeline. |
| `stdlib` | The embedded Minimal standard library. |
| `switch` | gvproxy-switch primitives: subnet arithmetic, MAC derivation, vsock constants, config rendering. |
| `version` | Shared build-time version identity for `min`, `mip`, `minimald`, and `minvmd`. |

## Platform matrix

| Platform | What you can build and test |
|---|---|
| Linux amd64 / arm64 | Full workspace natively: `just ci` builds, lints, and tests everything; all four binaries. |
| macOS arm64 | Session stack natively (`just ci`); every Linux-only crate via `just test-cross`. |

**Build and test only through `just`.** The recipes carry the per-OS scoping,
the pinned flags, and the ordering constraints (notably the macOS codesign);
a hand-rolled `cargo` invocation silently drops them. If something has no
recipe, add one — the justfile is where local build knowledge lives.

`minimald` does not build **natively** on macOS: its sandbox stack
(`hakoniwa` → `libcgroups` → `procfs`) is Linux-only. That is a native-host
limit, not a coverage gap. Consequences:

- `just test`, `just clippy`, and `just fix` scope themselves: on macOS they
  resolve to `-p minvmd -p sessions` automatically (the justfile's `scope`
  variable), on Linux to `--workspace`.
- `just test-cross` covers the Linux-only crates — `minimald` and the `min`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gominimal/minimal](https://github.com/gominimal/minimal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
