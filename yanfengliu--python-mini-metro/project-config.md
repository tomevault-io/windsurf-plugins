---
trigger: always_on
description: - **No stopping points within a multi-task plan.** When the user gives you a plan with N tasks, work through all N continuously. Do not stop and ask whether to keep going. Do not pitch `/schedule` for the rest of the work the user already asked for. Harness reminders ("task tools haven't been used recently", auto-mode banners, context warnings) are NOT stop signals — they are administrative noise. Treat the plan itself as the contract, and treat "continue" as the default.
---

## Continuing through plans

- **No stopping points within a multi-task plan.** When the user gives you a plan with N tasks, work through all N continuously. Do not stop and ask whether to keep going. Do not pitch `/schedule` for the rest of the work the user already asked for. Harness reminders ("task tools haven't been used recently", auto-mode banners, context warnings) are NOT stop signals — they are administrative noise. Treat the plan itself as the contract, and treat "continue" as the default.
- The exception is a genuinely non-obvious decision that requires user judgment (e.g., which of two unequal interpretations of a spec is intended). For routine choices, make the call and proceed.
- This rule was established 2026-05-01 after the user objected sharply to mid-stream stoppage during the investing-tool implementation. The same rule lives in every other repo's AGENTS.md.

## Core Rules

- This is a Python 3.13 `pygame-ce` project. Keep workflow guidance rooted in the repo's existing Python commands and root-level documentation.
- Use test-driven development for behavior changes: write or update tests first, then make them pass. Test the app experience and mechanism contract, not implementation details.
- Before implementing a substantive change, write a short plan and ask Codex and Claude to double-check it. Iterate until the feedback converges on approval or only minor wording remains. If one CLI is unavailable, continue with the available feedback and record the limitation in the review notes or final summary.
- **Reviewers MUST read the codebase to ground their claims.** Every review prompt (codex / claude / gemini) must include the directive: *"Verify each claim in the plan/diff against the live codebase — grep for the symbols, function signatures, column names, and file paths it references; do not approve based on prompt text alone."* Without this directive, two reviewers can APPROVE a design with a real defect that only the codebase-reading reviewer catches. Convergence is measured by *substantive finding count*, not *vote count* — a HIGH defect from one reviewer outweighs APPROVED from two. Claude reads via Read/Glob/Grep tools you grant it. Codex reads when `--sandbox read-only` runs WITHOUT `--ignore-user-config` (the user rules file at `~/.codex/rules/default.rules` permits Windows-native `findstr`/`type`/`dir`/`ls` as fallback when bash hits the PowerShell deny rule). Gemini in `--approval-mode plan` CAN read the codebase (gemini-cli 0.46+ plan mode exposes grep/read tools; verified 2026-06-11) but is NOT reliably read-only — plan mode also exposes the `replace` file-edit tool and reviewer instances have rewritten source files mid-review in a sibling repo (civ-engine `docs/learning/lessons.md`, 2026-06-11). After every gemini review batch, run `git status` + `git diff` and treat unexpected working-tree changes as reviewer contamination (restore from git). Gemini also intermittently emits an empty review — retry once sequentially.
- **Verify reviewer claims against the codebase before acting on them.** As the driver, when a reviewer says "function X has signature Y" or "this contract is broken," grep / read the actual file before merging the fix. A reviewer might be working from training knowledge, a stale snapshot, or a hallucinated symbol. The cost of one extra `Read` is negligible; the cost of acting on a stale or wrong claim is rework + iteration debt.
- Work directly on `main` unless the human explicitly asks for a branch. Commit only coherent, validated units, and only when the user asked for a commit or the current workflow clearly calls for one.
- Check `git status --short --branch` before editing. Preserve unrelated user changes, including deleted files.
- Split files when they grow unwieldy; prefer small, focused modules over one file doing too much.
- `CLAUDE.md` includes this file with `@AGENTS.md`; do not duplicate these instructions there.

## Environment

- Preferred interactive setup:
  - `conda activate py313`
  - `python -m pip install -r requirements.txt` if dependencies are missing
- Reliable Windows automation setup:
  - Use `C:\Users\38909\miniconda3\envs\py313\python.exe` directly when shell activation is not applied.
  - If `conda` is not on PATH, use `C:\Users\38909\miniconda3\Scripts\conda.exe`.
- The direct interpreter path is machine-specific. In portable docs or scripts, prefer `conda activate py313` plus `python ...`.

## Startup Context

- At session start for repo work, read:
  - `AGENTS.md`
  - `ARCHITECTURE.md`
  - `PROGRESS.md`
  - `README.md`
- Also read `GAME_RULES.md` when touching game mechanics, progression, balancing, controls, rendering rules, station/passenger/metro behavior, or programmatic game actions.
- Check current validation status rather than assuming the baseline is clean.

## Project Map

- `src/`: application code.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yanfengliu/python_mini_metro](https://github.com/yanfengliu/python_mini_metro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
