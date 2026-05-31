---
trigger: always_on
description: - **Build**: `./scripts/build-rust.sh dev|release|check|clean`
---

# Agent Coding Guidelines

## Quick Reference
- **Build**: `./scripts/build-rust.sh dev|release|check|clean`
- **Quality**: `./scripts/code-quality.sh fmt|clippy|audit|check`
- **Tests**: `cargo nextest run --all` (doctests: `cargo test --doc`)
- **Quality Gates**: `./scripts/quality-gates.sh`
- **Disk Cleanup**: `./scripts/clean-artifacts.sh [quick|standard|full] [--node-modules]`

Memory system: Rust/Tokio + Turso + redb + embeddings (OpenAI/Cohere/Ollama/local)
Crates: do-memory-core, do-memory-storage-turso, do-memory-storage-redb, do-memory-mcp, do-memory-cli, do-memory-test-utils, benches

## Skill + CLI Pattern (CRITICAL)
Always use Skill + CLI first for high-frequency ops:
| Operation | Skill | CLI |
|-----------|-------|-----|
| Build | `build-rust` | `./scripts/build-rust.sh` |
| Format/Lint | `code-quality` | `./scripts/code-quality.sh` |
| Tests | `test-runner` | `cargo nextest run --all` + `cargo test --doc` |
| Debug | `debug-troubleshoot` | - |

Before task tool: skill? → script? → Skill+CLI? → task tool?

## Change Workflow
1. Identify owner crate + module
2. Read existing patterns
3. Add/update tests
4. `./scripts/code-quality.sh fmt`
5. `./scripts/code-quality.sh clippy --workspace`
6. `cargo nextest run -p <crate>`
7. `cargo nextest run --all`
8. `cargo test --doc`
9. `./scripts/quality-gates.sh` (coverage threshold is `QUALITY_GATE_COVERAGE_THRESHOLD`, default 90)
10. `git status` - verify all changes staged

## Core Invariants (Never Break)
- **Async**: Tokio everywhere. No blocking (use `spawn_blocking`)
- **Storage**: Parameterized SQL only. Short transactions. No locks across `.await`
- **Serialization**: Postcard required (not bincode)
- **Clippy**: Zero warnings (`-D warnings`). Fix, don't suppress
- **Files**: ≤500 LOC per source file
- **Tests**: ≥90% coverage. `#[tokio::test]` for async. AAA pattern
- **Docs**: URLs wrapped in `<...>`. New types re-exported from `lib.rs`

## Documentation Rules
- Wrap URLs in angle brackets, re-export new public types from `lib.rs`, and run `cargo doc --no-deps --document-private-items` before commit

## Common Pitfalls

- **Coderabbitai review loops**: Always `read_files` on the target file before acting on a finding. Trust current code, not conversation summaries or cached search results. Fix history may not match current tree.
- Read patterns first; roadmap and status docs can lag real repo state.
- Verify release/package reality with `gh release view` and `cargo metadata` before editing version plans.
- Update `ROADMAP_ACTIVE.md`, `GOALS.md`, `ACTIONS.md`, `GOAP_STATE.md`, and `STATUS/CURRENT.md` together when sprint priorities change.
- For CPU/token work, use `goap-agent` first, then `agent-coordination`, then the implementation/validation skills.

Before implementing: Read 3+ source files, check ADRs

## Planning & Decisions
- **Use `goap-agent` skill** for complex tasks - decomposes into atomic goals
- **Use `agent-coordination`** when CPU/token or release/doc work can run in parallel
- **Check `plans/adr/`** for Architecture Decision Records before changes
- **Update `plans/ROADMAPS/ROADMAP_ACTIVE.md`** with progress
- **Keep `agent_docs/LESSONS.md` + `AGENTS.md` aligned** when recording non-obvious workflow learnings

## Tool Selection Enforcement

Target Bash:Grep ratio of 2:1 (current: 17:1)

**Use Grep for**:
- Finding files: `Grep pattern="*.rs"`
- Searching content: `Grep pattern="fn name"`
- Finding definitions: `Grep pattern="struct Name"`
- Checking usage: `Grep pattern="use crate"`

**Use Bash for**:
- File operations: `cp`, `mv`, `rm`
- Git commands: `git status`, `git diff`
- Running scripts: `./scripts/*.sh`
- Running workspace tests: `cargo nextest run --all`, `cargo test --doc`

**Before Bash**: Consider if Grep would be more efficient.

## Atomic Change Rules
1. **One change per commit** - message describes exactly what changed
2. **Workflow**: make change → test → quality check → verify → commit
3. **Format**: `feat(module): description`, `fix(module): description`
4. Never batch incomplete work

## Required Checks Before Commit
- [ ] `./scripts/code-quality.sh fmt`
- [ ] `./scripts/code-quality.sh clippy --workspace`
- [ ] `./scripts/build-rust.sh check`
- [ ] `cargo nextest run --all`
- [ ] `cargo test --doc`
- [ ] `cargo doc --no-deps --document-private-items` (catches bare URLs)
- [ ] `./scripts/quality-gates.sh` (coverage must be `>=90%`, unless threshold explicitly raised)
- [ ] `git status` - verify all changes staged

## Git Workflow
- **Branch Protection**: Direct pushes to `main` BLOCKED. Always work on a branch.
- See `agent_docs/git_workflow.md` for details.
Feature flags: `openai`, `local-embeddings`, `turso`, `redb`, `embeddings-full`, `full`, `csm`

## CSM Integration

Enable CPU-local cascading retrieval with the `csm` feature flag:
```bash
cargo build --features csm
```

**Available types when enabled:**
- `Bm25Index` - First-tier keyword search (no API calls)
- `HVec10240` - 10,240-bit HDC vectors for similarity
- `ConceptGraph` - Ontology expansion for synonym matching
- `CascadeRetriever` - Tier escalation orchestration

**Docs**: `agent_docs/csm_integration.md` for full cascade pipeline (WG-128 through WG-131).

## Security
- Use env vars (never hardcode)
- Parameterized SQL

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [d-o-hub/rust-self-learning-memory](https://github.com/d-o-hub/rust-self-learning-memory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->
