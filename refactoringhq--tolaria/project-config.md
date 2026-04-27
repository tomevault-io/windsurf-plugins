---
trigger: always_on
description: > Quick links: [Architecture](docs/ARCHITECTURE.md) · [Abstractions](docs/ABSTRACTIONS.md) · [Wireframes](ui-design.pen)
---

# AGENTS.md — Tolaria App

> Quick links: [Architecture](docs/ARCHITECTURE.md) · [Abstractions](docs/ABSTRACTIONS.md) · [Wireframes](ui-design.pen)

---

## 1. Task Workflow

### 1a. Pick up a task

Run `/laputa-next-task` — fetches next task (To Rework first, then Open), moves to In Progress, returns full description.

**Before writing a single line of code:** run `mcp__codescene__code_health_score` to check the current codebase health against `.codescene-thresholds`. If the score is already below the threshold, **stop and refactor first** — find the worst files with the MCP, improve them, commit, then start the task. Never start feature work on a codebase that is already below the gate.

- Read task description and all comments fully
- For To Rework: the ❌ QA failed comment tells you exactly what to fix
- Check `docs/adr/` for relevant architecture decisions before structural choices
- Add a comment: `🚀 Starting work on this task. [Brief description of approach]`

### 1b. Implement

- Work on `main` branch — **no branches, no PRs, ever**. Pre-commit and pre-push block work from any other branch.
- Commit every 20–30 min: `feat:`, `fix:`, `refactor:`, `test:`, `docs:`
- **⛔ NEVER use --no-verify**
- For UI tasks: open `ui-design.pen` first, study visual language, design in light mode

### 1c. When done

**Phase 1 — Playwright (only for core user flows):**

Write Playwright test in `tests/smoke/<slug>.spec.ts` only if feature touches: vault open, note create/save/delete, search, wikilink navigation, git commit/push, conflict resolution. Tag a test with `@smoke` only if it protects a core pre-push workflow. Do NOT tag cosmetic or mock-heavy checks — keep those in the full regression lane. The curated `pnpm playwright:smoke` suite must stay under **5 minutes**; use `pnpm playwright:regression` for the full Playwright pass.

```bash
pnpm dev --port 5201 &
sleep 3
BASE_URL="http://localhost:5201" npx playwright test tests/smoke/<slug>.spec.ts
```

**Phase 2 — Native app QA:**

```bash
pnpm tauri dev &
sleep 10
bash ~/.openclaw/skills/tolaria-qa/scripts/focus-app.sh laputa
bash ~/.openclaw/skills/tolaria-qa/scripts/screenshot.sh /tmp/qa-native.png
```

Use `osascript` for keyboard interactions. Write result as Todoist comment (✅ or ❌). **⚠️ WKWebView:** `osascript keystroke` blocked inside editor — rely on Playwright for text input features.

After both phases pass, add a **completion comment** to the Todoist task before running `/laputa-done`. The comment must include:
- What was implemented (1–2 lines)
- QA: what was tested and how (Playwright / native screenshot / osascript)
- Refactoring: any files refactored to meet the CodeScene gate (or "none needed")
- ADRs: any new/updated ADRs (or "none")
- Docs: any updated docs (ARCHITECTURE.md, ABSTRACTIONS.md, etc.) (or "none")
- Code health: final Hotspot and Average scores after push

Then run `/laputa-done <task_id>` → moves to In Review, notifies Brian, self-dispatches next task.

---

## 2. Development Process

### Commits & pushes

- Push directly to `main` — no PRs, no branches. Pre-push blocks non-`main` pushes.
- Pre-push hook runs full check suite (build + tests + core Playwright smoke + CodeScene)
- **A task is NOT done until `git push origin main` succeeds.** If the hook blocks: read the error, fix it (clippy, tests, CodeScene, build), commit the fix, push again. **⛔ NEVER use --no-verify**

### TDD (mandatory)

Red → Green → Refactor → Commit. One cycle per commit. For bugs: write failing regression test first, then fix. Exception: pure CSS/layout changes.

**Test quality (Kent Beck's Desiderata):** Isolated · Deterministic · Fast · Behavioral · Structure-insensitive · Specific · Predictive. Fix flaky tests first. Prefer E2E over unit tests for user flows.

### Code health (mandatory)

Pre-commit and pre-push hooks enforce **Hotspot Code Health** and **Average Code Health** ≥ thresholds in `.codescene-thresholds`. Both gates block commit/push. Thresholds are a **ratchet** — only go up. When pre-push sees improved remote scores, it updates `.codescene-thresholds`, stages it, and stops so you can commit the new floor with normal verified hooks before pushing again. Never add `// eslint-disable`, `#[allow(...)]`, or `as any`.

**⛔ NEVER edit `.codescene-thresholds` to lower the values.** If the gate blocks you, improve the code — do not lower the bar.

**CodeScene access order:** use CodeScene MCP tools if available. If MCP is unavailable, use the installed `cs` CLI for file-level review/delta work, and use the CodeScene API (`CODESCENE_PAT` + `CODESCENE_PROJECT_ID`) for project-wide Hotspot/Average threshold checks from `.codescene-thresholds`.

**Before editing any existing code file:** capture its current file-level CodeScene score. After your edits, re-run the same file-level review and verify the score is higher. If the file already starts at `10.0`, it must remain `10.0`.

**New files:** every new **scorable code file** must reach CodeScene score `10.0` before commit. If CodeScene reports `null` / "no scorable code" for a new file, it must still have zero CodeScene findings/warnings.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [refactoringhq/tolaria](https://github.com/refactoringhq/tolaria) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
