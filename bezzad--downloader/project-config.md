---
trigger: always_on
description: Guidance for working in this repository.
---

# CLAUDE.md

Guidance for working in this repository.

## Workflow & progress tracking

- **Work directly on `develop`.** Do ALL work on the `develop` branch. Never create
  feature branches.
- **Commit frequently** — one commit per logical step, with clear messages — and **push
  to `develop`** so any machine can pull the latest state.
- **Never strand work.** If work is unfinished at the end of a session, commit WIP to
  `develop` anyway with a `wip:` message prefix so nothing is stuck on one machine.
- **Maintain `PLAN.md` at all times** as the source of truth:
  - When given tasks, write them into **Todo** first, then start.
  - Move a task to **Active** and mark it `[~]` when you start it.
  - Mark it `[x]` and move it to **Done** with a one-line note and the commit hash when
    finished.
  - Mark it `[!]` and move it to **Blocked/Failed** with the reason if it fails.
  - Update **"Last updated"** and **"Now working on"** each time.
- **Commit `PLAN.md` together with the code change it describes**, on `develop`.
- **For large backlogs, also keep `TASKS.md` updated** as the full board.
- **At the START of every session, read `PLAN.md` (and `TASKS.md`) and continue from
  there.** Never rely on in-session memory surviving across machines — if it matters, it
  must be in `PLAN.md` and committed.

## Packaging

- **Always pack the NuGet package in `Release` mode, output to `src/Downloader/bin/nupkg/`.**
  This is the project's standard package output location.

  ```bash
  dotnet pack src/Downloader/Downloader.csproj -c Release -o src/Downloader/bin/nupkg/
  ```

  This produces `Downloader.<version>.nupkg` (and the symbols package) there. Use this path
  for any local pack/publish step instead of a temp directory.

## Token-efficient builds & tests (MANDATORY)

- **`dotnet build`**: always run with `-v q --nologo`. Only re-run without `-v q` if you need
  to inspect a specific error in detail.
- **`dotnet test`**: always run with `-v q --nologo`. On failure, re-run ONLY the failing
  test(s) with `--filter FullyQualifiedName~<TestName>` instead of the whole suite.
- **Long-running commands** (`dotnet test`, `dotnet build`, `dotnet pack`, `gh run watch`):
  run them with `run_in_background: true` and wait for the completion notification — never
  poll in a `while … sleep` loop, and never dump their full output into context. After
  completion, read only the tail / failure section of the output.

<!-- rtk-instructions v2 -->
# RTK (Rust Token Killer) - Token-Optimized Commands

## Golden Rule

**Always prefix commands with `rtk`**. If RTK has a dedicated filter, it uses it. If not, it passes through unchanged. This means RTK is always safe to use.

**Important**: Even in command chains with `&&`, use `rtk`:
```bash
# ❌ Wrong
git add . && git commit -m "msg" && git push

# ✅ Correct
rtk git add . && rtk git commit -m "msg" && rtk git push
```

## RTK Commands by Workflow

### Build & Compile (80-90% savings)
```bash
rtk cargo build         # Cargo build output
rtk cargo check         # Cargo check output
rtk cargo clippy        # Clippy warnings grouped by file (80%)
rtk tsc                 # TypeScript errors grouped by file/code (83%)
rtk lint                # ESLint/Biome violations grouped (84%)
rtk prettier --check    # Files needing format only (70%)
rtk next build          # Next.js build with route metrics (87%)
```

### Test (60-99% savings)
```bash
rtk cargo test          # Cargo test failures only (90%)
rtk go test             # Go test failures only (90%)
rtk jest                # Jest failures only (99.5%)
rtk vitest              # Vitest failures only (99.5%)
rtk playwright test     # Playwright failures only (94%)
rtk pytest              # Python test failures only (90%)
rtk rake test           # Ruby test failures only (90%)
rtk rspec               # RSpec test failures only (60%)
rtk test <cmd>          # Generic test wrapper - failures only
```

### Git (59-80% savings)
```bash
rtk git status          # Compact status
rtk git log             # Compact log (works with all git flags)
rtk git diff            # Compact diff (80%)
rtk git show            # Compact show (80%)
rtk git add             # Ultra-compact confirmations (59%)
rtk git commit          # Ultra-compact confirmations (59%)
rtk git push            # Ultra-compact confirmations
rtk git pull            # Ultra-compact confirmations
rtk git branch          # Compact branch list
rtk git fetch           # Compact fetch
rtk git stash           # Compact stash
rtk git worktree        # Compact worktree
```

Note: Git passthrough works for ALL subcommands, even those not explicitly listed.

### GitHub (26-87% savings)
```bash
rtk gh pr view <num>    # Compact PR view (87%)
rtk gh pr checks        # Compact PR checks (79%)
rtk gh run list         # Compact workflow runs (82%)
rtk gh issue list       # Compact issue list (80%)
rtk gh api              # Compact API responses (26%)
```

### JavaScript/TypeScript Tooling (70-90% savings)
```bash
rtk pnpm list           # Compact dependency tree (70%)
rtk pnpm outdated       # Compact outdated packages (80%)
rtk pnpm install        # Compact install output (90%)
rtk npm run <script>    # Compact npm script output
rtk npx <cmd>           # Compact npx command output

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bezzad/Downloader](https://github.com/bezzad/Downloader) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
