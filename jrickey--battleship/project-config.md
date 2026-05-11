---
trigger: always_on
description: PC port of Super Smash Bros. 64 built from the complete decompilation at github.com/VetriTheRetri/ssb-decomp-re. Target integration: libultraship (LUS) + Torch asset pipeline.
---

# SSB64 PC Port — Claude Session Context

PC port of Super Smash Bros. 64 built from the complete decompilation at github.com/VetriTheRetri/ssb-decomp-re. Target integration: libultraship (LUS) + Torch asset pipeline.

## Documentation

Detailed reference material lives under `docs/`. Read the file that matches the task before touching code. When looking for a topic not listed here, run `ls docs/` and `ls docs/bugs/` to see what's available.

| Topic | File |
|-------|------|
| Project status, ROM info, dependencies, source tree layout | `docs/architecture.md` |
| C type system, decomp naming prefixes, code style, macros | `docs/c_conventions.md` |
| RDRAM / RSP / RDP / GBI / audio / threading / controller / endianness | `docs/n64_reference.md` |
| CMake build, reloc stub regen, runtime logs, LP64 compat notes | `docs/build_and_tooling.md` |
| GBI trace capture (port + M64P plugin) and `gbi_diff.py` usage | `docs/debug_gbi_trace.md` |
| IDO BE bitfield layout audit (compile + rabbitizer disasm to verify port struct bit positions) | `docs/debug_ido_bitfield_layout.md` |
| Resolved bugs (index + per-bug root cause / fix write-ups) | `docs/bugs/README.md` |

Ongoing investigations and handoff notes are loose `.md` files at the top level of `docs/` — check there before starting work on rendering, collision, or animation issues so you don't duplicate prior effort.

When you fix a new significant bug, add an entry under `docs/bugs/` using the slug pattern `<topic>_<YYYY-MM-DD>.md` and link it from `docs/bugs/README.md`.

---

## Parallel Sessions — Worktree Workflow

Multiple Claude windows working in the same checkout will clobber each other's source edits and build outputs. **Every parallel session works in its own git worktree.**

### Spinning up a new worktree

```bash
./scripts/new-worktree.sh <slug>           # configure only (fast)
./scripts/new-worktree.sh <slug> --build   # configure + full Debug compile
./scripts/new-worktree.sh <slug> --base some-branch --release
```

Output lands at `.claude/worktrees/<slug>` on branch `agent/<slug>`. The script:
1. Creates the worktree and branch.
2. Symlinks `baserom.us.z64` (gitignored, too large to duplicate).
3. **Independently clones `libultraship` and `torch`** from the main tree's local submodule checkout (picks up pinned SHAs that may not be pushed to the forks yet), then resets each submodule's `origin` to whatever URL the main tree's submodule uses — usually SSH so pushes work.
4. Regenerates gitignored codegen (`reloc_data.h`, `yamls/us/reloc_*.yml`, credits encodings).
5. Runs `cmake -B build` inside the worktree (and compiles if `--build` given).

### What this gives you

- **Full edit authority everywhere** — any file under `src/`, `port/`, `libultraship/`, `torch/` is fair game. Submodule checkouts are real independent clones, not symlinks.
- **Zero collision** with other windows on source, build artifacts, or submodule state.
- **Normal git flow for submodule changes**:
  1. Edit and commit inside `<worktree>/libultraship/` (or `torch/`).
  2. Push to the fork: `git -C <worktree>/libultraship push origin ssb64` — goes to `JRickey/libultraship` on GitHub.
  3. In the outer worktree, bump the submodule pointer: `git add libultraship && git commit -m "Bump libultraship: <summary>"`.
  4. When the outer branch lands on main, the pointer update goes with it.

### Merging back to main

The outer worktree is a normal branch (`agent/<slug>`). Merge or PR it into `main` like any other branch. Submodule pointer bumps ride along in the commits.

### Cleanup

```bash
git worktree remove .claude/worktrees/<slug>
git branch -D agent/<slug>
```

Stale worktrees under `.claude/worktrees/` from past sessions are fine to remove — check `git worktree list` and prune anything you don't recognize.

### Gotchas

- **Never use relative `build` paths in Bash tool calls** — Claude Code resets cwd between `Bash` calls. `cmake --build build` from the project root builds the main tree, not the worktree. Always use absolute paths: `cmake --build <worktree>/build ...`.
- `build.sh --skip-extract` still tries `git submodule update --init` unconditionally, which fails on a new worktree (pinned SHAs not on remote). Use `new-worktree.sh` instead, or drive `cmake`/`cmake --build` directly with absolute paths.
- The binary loads `BattleShip.o2r`, `f3d.o2r`, and `ssb64.o2r` from its CWD at launch; without them it exits with `archive ... does not exist`. `new-worktree.sh` symlinks all three from the main tree's `build/` into the worktree's `build/`. If the main tree has never been extracted, run `./build.sh` there first so the symlinks resolve.

---

## Agent Directives

### Pre-Work

1. **THE "STEP 0" RULE**: Before any structural refactor on a file >300 LOC, first remove dead code, unused exports, unused imports, and debug logs. Commit cleanup separately.

2. **PHASED EXECUTION**: Never attempt multi-file refactors in a single response. Break work into phases. Complete Phase 1, run verification, wait for approval before Phase 2. Max 5 files per phase.

### Code Quality


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JRickey/BattleShip](https://github.com/JRickey/BattleShip) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-01 -->
