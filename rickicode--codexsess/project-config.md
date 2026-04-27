---
trigger: always_on
description: - For any UI/UX/design request, always use `Uncodixfy.md` as the single design reference.
---

# AGENTS.md

## Design Source Of Truth (Mandatory)

- For any UI/UX/design request, always use `Uncodixfy.md` as the single design reference.
- Do not introduce additional design guides; just mention and follow `Uncodixfy.md`.

## Session Change Tracking (Mandatory)

- Before editing, run `rtk git status --short` to capture baseline.
- After editing, run `rtk git diff --name-only` and `rtk git diff` to review exactly what changed.
- Before finishing a session, run `rtk git status --short` again and summarize changed files.
- If a commit is requested, include a concise scope in commit message and ensure only intended files are staged.
- Prefer these commands for traceability:
  - `rtk git status --short`
  - `rtk git diff --name-only`
  - `rtk git diff`
  - `rtk git show --stat`

## Implementation Log (Mandatory)

- Every implementation session must update [`docs/implementation-log.md`](docs/implementation-log.md).
- Write only changes that were actually implemented, not plans or intentions.
- Add a new dated entry whenever code, behavior, runtime plumbing, persistence, tests, or operational flow changes.
- Each entry must state:
  - scope
  - files or subsystems touched
  - behavior/runtime effect
  - validation status
  - open follow-up items, if any
- Keep entries append-only and ordered newest first so the latest implementation state is obvious.
- If the work is partial, state clearly what is done and what is not done yet.

## Test Timeout Policy (Mandatory)

- Every test command must include an explicit timeout to avoid long-running hangs and unnecessary RAM consumption.
- Preferred patterns:
  - `rtk timeout 120s <test-command>`
  - or native test timeout flags (for example `go test -timeout 120s`, `vitest --testTimeout=120000`).
- For large suites, start with targeted tests plus timeout before running broader suites.

## Changelog Update (Mandatory)

- Update `CHANGELOG.md` only for significant, user-visible product behavior changes.
- Changelog scope only covers major changes in:
  - Go backend/runtime code (for example `main.go`, `internal/**/*.go`)
  - Frontend application source (for example `web/src/**`)
- Do not add changelog entries for:
  - docs or markdown-only edits (`*.md`)
  - workflow/packaging/scripts/assets-only edits
  - small cosmetic-only UI tweaks (for example minor font/spacing/color adjustments)
- If no significant product behavior change happened, skip changelog update.

### Backend + frontend dev

- `make dev` — runs backend hot-reload via `air` on port 3078 and starts the main `frontend` dev server.
- `make run` — builds then runs the `hijinetbill` binary.
- `make kill` — stops the binary and frees ports 3077/1812/1813.


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

### Test (90-99% savings)

```bash
rtk cargo test          # Cargo test failures only (90%)
rtk vitest run          # Vitest failures only (99.5%)
rtk playwright test     # Playwright failures only (94%)
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

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rickicode/CodexSess](https://github.com/rickicode/CodexSess) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
