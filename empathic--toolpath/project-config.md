---
trigger: always_on
description: Toolpath is a format for artifact transformation provenance. It records who changed what, why, what they tried that didn't work, and how to verify all of it. Think "git blame, but for everything that happens to code, including the stuff git doesn't see."
---

# CLAUDE.md

## What is this project?

Toolpath is a format for artifact transformation provenance. It records who changed what, why, what they tried that didn't work, and how to verify all of it. Think "git blame, but for everything that happens to code, including the stuff git doesn't see."

Three core objects: **Step** (a single change), **Path** (a sequence of steps, e.g. a PR), **Graph** (a collection of paths, e.g. a release). Steps form a DAG via parent references. Dead ends are implicit -- steps not on the ancestry of `path.head`.

## Repository layout

```
Cargo.toml                      # workspace root (edition 2024, resolver 2)
crates/
  toolpath/                     # core types, builders, serde, query API
  toolpath-convo/               # provider-agnostic conversation types, traits, and ConversationView -> Path derivation
  toolpath-git/                 # derive from git repos (git2)
  toolpath-github/              # derive from GitHub pull requests (REST API)
  toolpath-claude/              # derive from Claude conversation logs
  toolpath-gemini/              # derive from Gemini CLI conversation logs
  toolpath-codex/               # derive from Codex CLI rollout files
  toolpath-copilot/             # derive from + project to GitHub Copilot CLI session logs (preview)
  toolpath-opencode/            # derive from opencode SQLite databases
  toolpath-cursor/              # derive from Cursor (IDE) state.vscdb bubble store
  toolpath-pi/                  # derive from Pi (pi.dev) agent session logs
  toolpath-dot/                 # Graphviz DOT rendering
  toolpath-md/                  # Markdown rendering for LLM consumption
  path-cli/                     # unified CLI (binary: path)
  toolpath-cli/                 # deprecated shim that re-exports path-cli (excluded from the workspace; see below)
  pathbase-client/              # progenitor-derived client for the Pathbase HTTP API
                                # (spec at crates/pathbase-client/openapi.json; refresh via scripts/refresh-pathbase-openapi.sh)
.claude-plugin/
  marketplace.json              # Claude Code plugin marketplace (add with `/plugin marketplace add empathic/toolpath`)
plugins/
  claude-code/                  # Claude Code plugin "path": /path:share + /path:query, bundles the CLI
docs/agents/formats/            # format references for the agent on-disk formats we derive from
schema/toolpath.schema.json     # JSON Schema for the toolpath format
examples/*.json                 # example documents (step, path, graph)
RFC.md                          # full format specification
FAQ.md                          # design rationale, FAQ, and open questions
```

## Dependency graph

```
path-cli (binary: path)
 ├── toolpath           (core types)
 ├── toolpath-convo   → toolpath (conversation abstraction + shared derivation)
 ├── toolpath-git     → toolpath
 ├── toolpath-github  → toolpath
 ├── toolpath-claude  → toolpath, toolpath-convo
 ├── toolpath-gemini  → toolpath, toolpath-convo
 ├── toolpath-codex   → toolpath, toolpath-convo
 ├── toolpath-copilot → toolpath, toolpath-convo  (preview)
 ├── toolpath-opencode → toolpath, toolpath-convo
 ├── toolpath-cursor  → toolpath, toolpath-convo
 ├── toolpath-pi      → toolpath, toolpath-convo
 ├── toolpath-dot     → toolpath
 └── toolpath-md      → toolpath

pathbase-client      (no toolpath deps; built from crates/pathbase-client/openapi.json)

toolpath-cli (deprecated shim, binary: path)
 └── path-cli
```

## Build and test

```bash
cargo build --workspace
cargo test --workspace
cargo clippy --workspace -- -D warnings
```

Those three commands are the inner loop, not the gate. Before a branch is called ready for review, run the full gate set. It is the same script CI's `ci` job runs, so a subset of it proves nothing about CI:

```bash
scripts/quality_gates.sh          # or: just ci; --verbose streams each gate's output
```

Requires Rust 1.85+ (edition 2024). Pinned to 1.94.0 via `rust-toolchain.toml`.

If `cargo` is not on your PATH, `flake.nix` carries a devShell with everything the
justfile and `scripts/quality_gates.sh` assume — the Rust toolchain plus shellcheck,
node/pnpm, jq, curl and fzf, with openssl wired up for `openssl-sys`:

```bash
nix develop                                   # or: nix develop --command <cmd>
nix develop --command ./scripts/quality_gates.sh
```

The shell's Rust comes from nixpkgs and is **ahead of** the 1.94.0 pin — `rust-toolchain.toml`
is read by rustup, which the shell does not provide. Clippy gains lints between releases, so
green in the shell is evidence, not proof; the pinned toolchain is the real gate.

The same flake builds the binary and exports a home-manager module, so a nix consumer can
take this repo as a flake input and follow a ref of it instead of pinning a rev by hand:

```bash
nix build .#toolpath          # → result/bin/path; version read from crates/path-cli/Cargo.toml
# programs.toolpath.{enable,package,devBin} via homeManagerModules.toolpath (modules/toolpath.nix)
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [empathic/toolpath](https://github.com/empathic/toolpath) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
