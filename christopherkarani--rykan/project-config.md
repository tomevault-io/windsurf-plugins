---
trigger: always_on
description: - Verify the real checkout and files before changing anything.
---

# Orca Agent Instructions

## Working Style

- Verify the real checkout and files before changing anything.
- Use TDD for non-trivial changes.
- Keep edits surgical and tied to the request.
- **Act by default** when the task is clear: local edits, tests, implement/fix loops, and sub-agent spawns do not need human approval. Pause only for ambiguity, architecture forks, or irreversible/shared actions (push, force-push, published PR reviews, deletes the user did not ask for, new dependencies, anything that changes shared remote state).
- **Direct work** for small or mechanical tasks (typos, one-liners, pure formatting, trivial renames).
- **Prefer sub-agents** for non-trivial multi-step work. Also use sub-agents for parallel independent work, blast-radius isolation (worktrees), Zig specialist lanes, or explicit orchestration.
- When multi-step, keep a short internal plan (todos). That is not a request for human go-ahead.
- Sub-agent output is **advisory until the main agent verifies** it against the tree, tests, and product rules.
- Full work + review SOP: [`docs/agents/work-and-review.md`](docs/agents/work-and-review.md).

## Repo Boundaries

- Treat this repository as public-facing by default.
- Keep local planning, handoffs, reviews, and task notes out of tracked docs unless the user explicitly asks to publish them.
- Keep session-local artifacts in `planning/`; only `planning/README.md` is tracked.
- Before staging or committing, run:

```bash
git ls-files | rg '(^planning/|^go_to_market/|^customer_pilot/|^tasks/|^reports/|^\\.orca-edge/|^\\.edge/|^dist/|^dist-dry-run/|^docs/release/|^docs/orca_opencode_openclaw_plan/|node_modules/)'
```

- Never commit generated release archives, SBOMs, checksums, dry-run package output, red-team replay output, customer-pilot templates, SOW/NDA notes, target-account templates, outreach copy, pricing guidance, or task-memory logs.

## Orca Context

- Zig is the primary (and sole) user-facing CLI and shell evaluator.
- Shell command security decisions are owned by the in-process Zig `shell_engine`. `ORCA_SHELL_EVAL=rust` is rejected (legacy Rust daemon Evaluate removed); production path is Zig only.
- Non-shell events (files, network, MCP/tools, effects) stay on the Zig policy path.
- Do not reintroduce a required Rust daemon for hook/run/shim shell gating.
- Shell evaluator internal errors fail closed with `deny`.

## Toolchain

- Use Zig **0.16.0** (`.zigversion`).
- Prefer `./scripts/zig` over bare `zig` / `zig build` / `zig build test`.
- If a Zig command fails and the version is wrong, fix the toolchain first:

```bash
./scripts/ensure-zig-toolchain.sh --install
eval "$(./scripts/ensure-zig-toolchain.sh --export)"   # or: direnv allow
./scripts/zig version   # must print 0.16.0
```

- The former `orca-rs/` Rust daemon crate has been removed from the product tree. Do not add Cargo to `zig build`.

## Verification gates (read this before every verify loop)

**Default: use the narrowest gate that can catch your change.**
“test-fast” means *fast relative to the full suite*, not “seconds.” The Zig lib unit binary alone is often **several minutes** (~1.1k tests via the monopath `src/root.zig` graph).

**One-shot path picker for agents:**

```bash
./scripts/agent-gate.sh                  # auto from git dirty paths
./scripts/agent-gate.sh --dry-run        # print selection only
./scripts/agent-gate.sh units            # force L1
./scripts/agent-gate.sh --paths src/cli/plugin.zig
./scripts/zig build test-shell-engine    # Zig shell evaluator + MVP corpus
```

### Tier ladder

| Tier | When | Command | Typical cost |
|------|------|---------|--------------|
| **L0 compile** | After every Zig edit; “does it compile?” | `./scripts/compile-fast.sh check` | ~seconds–tens of seconds |
| **L0.5 shell** | `src/shell_engine/**` | `./scripts/zig build test-shell-engine` | seconds |
| **L0.5 domain** | Edits confined to one domain | `./scripts/test-slice.sh sandbox\|policy\|intercept` | often **seconds–tens of seconds** |
| **L1 units** | Broad Zig logic needs unit confidence | `./scripts/test-fast.sh units` **or** monopath `test-lib` | **multi-minute** monopath |
| **L2 product** | Policy/CLI handoff, pre-PR light gate | `./scripts/test-fast.sh` (or `full`) | L1 + ~1s quick-install matrix |
| **L3 full Zig** | Pre-merge / full suite (single CI job) | `./scripts/zig build test` | full phase/plugin/fuzz suites |
| **L4 pre-merge** | Explicit pre-merge only | `./scripts/verify-pre-merge.sh` | L2 + L3 + install/uninstall regressions |

### Path → gate matrix

| Touched paths | Prefer |
|---------------|--------|
| `src/shell_engine/**` | **L0.5** `./scripts/zig build test-shell-engine` |
| Single Zig file, compile check only | **L0** `./scripts/compile-fast.sh check` |
| `src/sandbox/**` only | **L0.5** `./scripts/test-slice.sh sandbox` / `agent-gate.sh` |
| `src/policy/**` only | **L0.5** `./scripts/test-slice.sh policy` |
| `src/intercept/**` only | **L0.5** `./scripts/test-slice.sh intercept` |
| `src/**`, `packages/**`, `build.zig` (broad logic) | L0 → **L1** `test-fast.sh units` or `agent-gate.sh` |
| `scripts/**` only | `bash -n` + the script’s own smoke; avoid L3 unless the script is a gate itself |

### Pitfalls (do not)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [christopherkarani/rykan](https://github.com/christopherkarani/rykan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
