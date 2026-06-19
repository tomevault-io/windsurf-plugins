---
trigger: always_on
description: >-
---


# bugsweep

A safe, auditable, autonomous bug-hunting pipeline. It separates **finding** a bug from
**challenging** it from **confirming** it from **fixing** it, so the model never
rubber-stamps its own guesses; it builds whole-repo context so it can catch large
cross-file bugs; it primes itself with anti-patterns common to the stack under review; and
it routes every irreversible git operation through deterministic shell scripts so the
safety guarantees never depend on the model's judgment. All progress is written to disk so
a long run can reset context and continue without losing work.

## The trust contract (read first — it governs everything)

Non-negotiable. Scripts in `scripts/` enforce the irreversible parts; you enforce the
rest. If a rule can't be honored, STOP and report — never work around it.

1. **Work only on a throwaway branch** (`bugsweep/<timestamp>`, created by preflight).
   Never commit to or switch the user onto their original branch.
2. **Core run never touches remotes.** During preflight, hunt, fix, and finalize: no
   push/pull/fetch, no PR, no merge. The human is the only merge gate. Post-finalize
   merge/push/delete actions are allowed only after an explicit approved continuation, or
   through the optional cleanup/merge-gate script the human configured.
3. **No destructive operations, ever.** No `git reset --hard` on user content, no force
   anything, no deleting files/dirs, no `rm -rf`, no history rewriting.
4. **Preserve the user's work.** Preflight stashes uncommitted changes; finalize restores
   them. Their starting branch and working tree end exactly as they began.
5. **One bug, one commit, auto-revert on regression.** Re-run checks after each fix; if a
   fix introduces ANY new failure, revert it and quarantine the bug.
6. **Fix only confirmed bugs** — findings must pass the full adversarial review first.
7. **Minimal surgical fixes only.** No refactoring, renaming, reformatting, or unrelated
   changes.
8. **Stay inside the caps** (iterations, runtime, fixes) and stop when converged.
9. **Everything is logged** to the run ledger so an overnight run is auditable.

The worst possible outcome of any run is a throwaway branch the user deletes. That is what
makes unattended autonomy safe.

## Modes

Parse the invocation; default to the SAFEST reading when ambiguous.

| Invocation | Behavior |
| --- | --- |
| `/bugsweep` | **Detect only.** Full pipeline, writes a report, no code changes. (Default.) |
| `/bugsweep --fix` | Find + adversarial-confirm + fix on the branch. Single pass. |
| `/bugsweep --approve` | Like `--fix`, but PAUSE for the user's OK before each fix. |
| `/bugsweep --autonomous` | Find + confirm + fix, then **loop** until clean or a cap, with periodic context checkpoints/resets. The unattended/overnight mode. Implies `--fix --loop`. |
| `/bugsweep <path>` | Scope to a file or directory (combine with any flag). |
| `/bugsweep --severity <low\|medium\|high\|critical>` | Only fix bugs at/above this severity. |
| `/bugsweep --update` | Update bugsweep to the latest version. Detects install location, runs `install.sh`, then exits. Re-invoke after updating. |

For unattended/overnight/"run all night"/fully autonomous behavior, use `--autonomous`.
Recommend a first-time `--approve` run to calibrate trust before `--autonomous`.

## Execution

### Step 0 — Preflight (deterministic safety setup)

**Version check (run this first, every invocation).** Detect the install location, compare
the local version against the published one, and handle `--update`:

```bash
# Locate the install — prefer Claude Code, fall back to Codex
_bs_dir=""
[ -d "$HOME/.claude/skills/bugsweep" ] && _bs_dir="$HOME/.claude/skills/bugsweep"
[ -z "$_bs_dir" ] && [ -d "$HOME/.codex/skills/bugsweep" ] && _bs_dir="$HOME/.codex/skills/bugsweep"

# Passive staleness check (non-blocking — a slow/offline network is silently ignored)
if [ -n "$_bs_dir" ]; then
  _bs_local=$(cat "$_bs_dir/VERSION" 2>/dev/null || echo "")
  _bs_remote=$(curl -sf --max-time 3 \
    https://raw.githubusercontent.com/shanemhamilton/bugsweep/main/VERSION 2>/dev/null || echo "")
  if [ -n "$_bs_local" ] && [ -n "$_bs_remote" ] && [ "$_bs_local" != "$_bs_remote" ]; then
    echo "⚠ bugsweep $_bs_remote is available (you have $_bs_local). Run /bugsweep --update to upgrade."
  fi
fi
```

**If `--update` was passed**, run the updater and stop — do not proceed to the hunt:
```bash
if [ -n "$_bs_dir" ]; then
  bash "$_bs_dir/install.sh"
  echo "✓ bugsweep updated. Re-invoke to start a fresh run on the new version."
else
  echo "✗ Could not locate bugsweep install (~/.claude/skills/bugsweep or ~/.codex/skills/bugsweep)."
  echo "  Re-install with: bash <(curl -fsSL https://raw.githubusercontent.com/shanemhamilton/bugsweep/main/install.sh)"
fi
# EXIT — do not run preflight or any hunt steps after --update
```

ALWAYS run preflight next, before reading any source file:
```bash
bash scripts/preflight.sh                     # detect / fix / approve modes
bash scripts/preflight.sh --mode autonomous   # when invoked with --autonomous
```
It verifies the repo is safe, refuses an unclean protected branch, stashes uncommitted
work, creates and checks out `bugsweep/<timestamp>`, and prints a `RUN_DIR` (under

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shanemhamilton/bugsweep](https://github.com/shanemhamilton/bugsweep) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
