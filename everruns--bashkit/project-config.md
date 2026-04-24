---
trigger: always_on
description: Telegraph. Drop filler/grammar. Min tokens.
---

## Coding-agent guidance

### Style

Telegraph. Drop filler/grammar. Min tokens.

### Critical Thinking

Fix root cause. Unsure: read more code; if stuck, ask w/ short options. Unrecognized changes: assume other agent; keep going. If causes issues, stop + ask.

### Principles

- Always make sure you are working on top of latest main from remote. Especially in worktrees: fetch `origin/main`, then rebase or recreate the worktree on top of it before editing.
- Important decisions as comments on top of file
- Code testable, smoke testable, runnable locally
- Small, incremental PR-sized changes
- No backward compat needed (internal code)
- Write failing test before fixing bug

### Specs

`specs/` contains feature specifications. New code should comply with these or propose changes.

| Spec | Description |
|------|-------------|
| architecture | Core interpreter architecture, module structure |
| parser | Bash syntax parser design |
| vfs | Virtual filesystem abstraction |
| testing | Testing strategy and patterns |
| builtins | Builtin command design (trait, ShellRef, ExecutionPlan) |
| security-testing | Fail-point injection for security testing |
| threat-model | Security threats and mitigations |
| parallel-execution | Threading model, Arc usage |
| documentation | Rustdoc guides, embedded markdown |
| release-process | Version tagging, crates.io + PyPI + npm publishing |
| implementation-status | Feature status, test coverage, limitations, POSIX compliance |
| tool-contract | Public LLM Tool trait contract |
| git-support | Sandboxed git operations on VFS |
| python-builtin | Embedded Python via Monty, security, resource limits |
| eval | LLM evaluation harness, dataset format, scoring |
| maintenance | Pre-release maintenance requirements |
| python-package | Python package, PyPI wheels, platform matrix |
| scripted-tool-orchestration | Compose ToolDef+callback pairs into OrchestratorTool via bash scripts |
| ssh-support | Sandboxed SSH/SCP/SFTP operations |
| zapcode-runtime | Embedded TypeScript via ZapCode, VFS bridging, resource limits |
| request-signing | Transparent Ed25519 request signing (bot-auth) per RFC 9421 |
| interactive-shell | Interactive REPL mode with rustyline line editing |

### Documentation

- **Public docs** live in `docs/` — user-facing articles (security, guides, etc.)
- **Rustdoc guides** live in `crates/bashkit/docs/` as markdown files
- Rustdoc guides embedded via `include_str!` (see `specs/documentation.md`)
- Edit `crates/bashkit/docs/*.md`, not the doc modules in `lib.rs`
- Add "See also" cross-links when creating new guides
- Run `cargo doc --open` to preview rustdoc changes

### Bashkit Principles

- All design decisions in `specs/` - no undocumented choices
- Everything runnable and testable - no theoretical code
- Don't stop until e2e works - verify before declaring done
- Examples tested in CI - must pass
- No silent deferral - `TODO` or `WTF` comment with explanation
- Verify crate assumptions before planning to use them

### Cloud Agent Setup

```bash
./scripts/init-cloud-env.sh   # Install just + gh
just build                    # Build project
```

#### Secrets via Doppler

`DOPPLER_TOKEN` is pre-configured. Fetch secrets with:

```bash
curl -s "https://api.doppler.com/v3/configs/config/secret?name=SECRET_NAME" \
  -u "$DOPPLER_TOKEN:" | python3 -c "import sys,json; print(json.load(sys.stdin)['value']['raw'])"
```

Common secrets: `GITHUB_TOKEN`, `ANTHROPIC_API_KEY`

For `gh` CLI, set `GH_TOKEN`:

```bash
export GH_TOKEN=$(curl -s "https://api.doppler.com/v3/configs/config/secret?name=GITHUB_TOKEN" \
  -u "$DOPPLER_TOKEN:" | python3 -c "import sys,json; print(json.load(sys.stdin)['value']['raw'])")
```

### Local Dev

```bash
just --list       # All commands
just build        # Build
just test         # Run tests
just check        # fmt + clippy + test
just pre-pr       # Pre-PR checks
```

### Rust

- Stable Rust, toolchain in `rust-toolchain.toml`
- `cargo fmt` and `cargo clippy -- -D warnings`
- License checks: `cargo deny check` (see `deny.toml`)

### Python

- Python package in `crates/bashkit-python/`
- Linter/formatter: `ruff` (config in `pyproject.toml`)
- `ruff check crates/bashkit-python` and `ruff format --check crates/bashkit-python`
- Tests: `pytest crates/bashkit-python/tests/ -v` (requires `maturin develop` first)
- CI: `.github/workflows/python.yml` (lint, test on 3.9/3.12/3.13, build wheel)

### Pre-PR Checklist

1. `just pre-pr` (runs 2-4 automatically)
2. `cargo fmt --check`
3. `cargo clippy --all-targets --all-features -- -D warnings`
4. `cargo test --all-features`
5. Unit tests cover both positive (expected behavior) and negative (error handling, edge cases) scenarios
6. Security tests if change touches user input, parsing, sandboxing, or permissions (see `specs/security-testing.md`)
7. Compatibility/differential tests if change affects Bash behavior parity (compare against real Bash)
8. Rebase on main: `git fetch origin main && git rebase origin/main` (for worktrees: verify the worktree `HEAD` is on latest `origin/main` before editing)
9. Update specs if behavior changes
10. CI green before merge
11. Resolve all PR comments
12. `cargo bench --bench parallel_execution` if touching Arc/async/Interpreter/builtins (see `specs/parallel-execution.md`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [everruns/bashkit](https://github.com/everruns/bashkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
