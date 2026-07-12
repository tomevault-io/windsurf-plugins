---
trigger: always_on
description: Harn is a programming language and runtime for orchestrating AI agents.
---

# AGENTS.md

Harn is a programming language and runtime for orchestrating AI agents.

Use this file for repo-local rules. Keep generated files generated. Prefer the
existing Harn runtime and stdlib over new host glue.

## Harn scripts

- Before writing or editing `.harn`, run `harn skill list --json`.
- Fetch the narrowest guide with `harn skill get <name> --full`.
- Use `harn-language` for syntax, modules, types, and `llm_call`.
- Use `harn-orchestration` for triggers, workers, personas, and agent loops.
- Fallback docs: `docs/llm/harn-quickref.md` and
  `docs/llm/harn-triggers-quickref.md`.
- Default new scripts to `fn main(harness: Harness) { ... }`.
- Route capabilities through `harness.*`.

Claude Code users get the same reminder through
`.claude/skills/harn-scripting/SKILL.md`; the full skill content ships in the
local `harn` binary so it matches the version in use.

## Setup

- Run `make setup` on a fresh clone. It installs hooks, optional developer
  tools, repo-local Node tooling when available, sccache config, per-worktree
  Cargo target config when `CODEX_WORKTREE_PATH` is set, and
  `cargo check --workspace`.
- Build caching is three layers, written into `.cargo/config.toml` by
  `scripts/dev_setup.sh`:
  1. **Shared `build-dir`** (`$TMPDIR/cargo-build-shared`, one per machine —
     `$TMPDIR` is per-user on macOS): Cargo 1.91+ splits intermediate
     artifacts from final ones, and Cargo's own fingerprinting dedupes
     registry-dep and build-script compilation across every worktree that
     shares the path. This is what gives cross-worktree caching. Older cargos
     ignore the key harmlessly. Override with `HARN_DEV_BUILD_DIR=<path>` at
     setup time, or escape-hatch a single invocation back to an isolated
     build dir with `CARGO_BUILD_BUILD_DIR=target/build`.
  2. **Per-worktree `target-dir`** (`$TMPDIR/harn-target/<parent>-<leaf>`):
     final binaries stay isolated per worktree, so concurrent sessions never
     swap the binary under test out from under each other.
  3. **sccache** (`rustc-wrapper`): caches same-path recompiles only. Its
     Rust hash is target-dir-path-dependent, so it does NOT dedupe
     compilation across different target dirs — the shared build-dir is what
     does.
  Orphaned per-worktree target dirs are reclaimed by
  `scripts/prune_stale_targets.sh` (run from setup at most daily); it never
  touches `cargo-build-shared`.
- Setup phases are fingerprinted under `.codex/dev-setup/`, so repeated setup
  is normally a fast no-op. Use `HARN_DEV_SETUP_FORCE=1 make setup` to refresh
  every phase.
- Codex app worktrees use `.codex/environments/environment.toml`, which calls
  `make setup` through the same repo bootstrap path.
- Claude Code project settings run `scripts/claude-dev-setup-once.sh` on
  session startup. It delegates to the same setup path once per dependency
  fingerprint and stores ignored logs under `.claude/dev-setup/`.
- Use `make test` for workspace Rust tests. It uses `cargo nextest` when
  available and falls back to `cargo test --workspace`.
- Use `make test-cargo` only when you need baseline Cargo behavior.
- Run `make install-hooks` if the repo hook path is not configured.
- Inspect CLI surface with `cargo run --quiet --bin harn -- --help`.
- The root `package.json` is repo tooling only. Portal, tree-sitter, and VS Code
  have their own package manifests.
- Repo-root portal scripts bootstrap `crates/harn-cli/portal/node_modules`, so
  `npm run portal:*` works in fresh worktrees.
- `crates/harn-wasm` is outside the Cargo workspace. Build it with
  `cd crates/harn-wasm && wasm-pack build`.

Keep installed hooks on. Pre-commit runs format/lint checks and generated-file
fixups. Pre-push runs targeted package checks, generated-file drift checks, and
affected Harn format/lint checks. Set `HARN_PREPUSH_FULL_TESTS=1` to run the
broader `make test` gate before pushing.

## Repository map

- `crates/harn-lexer`: tokenizer and spans.
- `crates/harn-parser`: AST, parser, and type checker.
- `crates/harn-stdlib`: embedded Harn stdlib source catalog.
- `crates/harn-vm`: compiler, VM, stdlib, providers, orchestration, transcripts,
  bridge, and ACP integration.
- `crates/harn-cli`: CLI, conformance runner, portal server, MCP/OAuth, A2A/ACP,
  replay, and eval tooling.
- `crates/harn-lint`, `crates/harn-fmt`: linting and formatting.
- `crates/harn-lsp`, `crates/harn-dap`: editor and debugger integrations.
- `crates/harn-cli/portal/`: React/Vite persisted-run UI.
- `conformance/tests/`: executable language/runtime spec.
- `spec/chapters/*.md`: canonical language spec, one file per section
  (`spec/HARN_SPEC.md` is the generated single-file assembly).
- `docs/src/`: Markdown docs (Diataxis IA), rendered by the `website/` site.
- `website/`: Vite + React + Tailwind site for harnlang.com; builds to `docs/dist/`.
- `tree-sitter-harn/`: tree-sitter grammar and tests.
- `editors/vscode/`: VS Code extension.

## Prompt templates

The `.harn.prompt` / `.prompt` engine used by `render(...)`,
`render_prompt(...)`, and `template.render` lives in
`crates/harn-vm/src/stdlib/template.rs`. Do not add another parser or evaluator.
Host-call and script-call rendering both go through `render_template_result`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [burin-labs/harn](https://github.com/burin-labs/harn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-12 -->
