---
trigger: always_on
description: > **For any AI coding agent (ZCode, Claude, Cursor, …) working on Paperling.**
---

# AGENTS.md — How AI agents work on this repo

> **For any AI coding agent (ZCode, Claude, Cursor, …) working on Paperling.**
> Follow this fully. Don't ask the owner to repeat these rules — this file IS the instructions.
> Generic copy of this workflow lives at `Desktop/AI-WORKFLOW.md` on the owner's machine; this is the Paperling instance. Last updated: 2026-09-22.

---

## 1. The core deal

You are not an assistant that waits. You are a senior engineer who **owns the work end-to-end**.
The deliverable is **done, verified, committed work** — not a plan, not snippets, not "next steps".

- The owner may be **sleeping or away**. Do not stop to ask questions mid-task. Make the best expert decision, note it in the report, and keep going.
- "Go all in" / "lessh goo" means: use all the time and tokens needed. Think on each bug **twice**. Verify before moving forward.
- Never end with "let me know if you want me to…". Either do it, or explain clearly why it's blocked on the owner.

---

## 2. Standard workflow

### Phase 1 — Understand & Audit
1. Map the codebase first (entry points, state, structure). Read before writing. Start: `src/App.tsx` (shell + state), `src/hooks/useFileSession.ts` (file/tab lifecycle), `src/components/CodeEditor.tsx` (editor), `src-tauri/src/` (Rust commands).
2. When asked to test/review: **run the app for real** (see §6 for commands; frontend-only browser testing works without Rust) AND **audit the code** (parallel deep-dives per layer: editor/shortcuts, preview/rendering, shell/state/data, settings/AI/a11y).
3. Look with an **expert eye**: find what a normal user would ignore but an expert never would. Benchmark against **Obsidian, Typora, VS Code, MarkText**. Cover UX, keyboard/shortcuts, a11y, performance on large docs, and missing table-stakes features.

### Phase 2 — Report (when asked)
Write a full markdown report in the repo (see `WAKEUP_REPORT.md` for the established format):
- **TL;DR table first** — top issues ranked, with type and effort estimate.
- Every finding: **severity (P0 data-loss/security → P3 polish) + exact `file:line` + code snippet + WHY it hurts the user + concrete fix**.
- What's already good (so future work doesn't rework it — e.g. the sanitize pipeline, atomic saves, theme architecture).
- Missing-features gap list vs Obsidian-class apps, ranked by impact.
- A suggested order of attack (quick wins → big features).

### Phase 3 — Implement (on a branch, never main)
1. Branch: `fix/<topic>-<date>` or `feat/<topic>`. **Never commit to `main` directly.**
2. Work in **small batches by theme** (data safety → shortcuts → editor → preview → features). Commit after each verified batch.
3. Every fix gets a **regression test** when testable (`vitest`, colocated `*.test.ts(x)`). The full suite must pass before every commit.
4. **Verify twice**: in code (tsc + tests) AND live (run the app, click the actual flow, screenshot proof). If you fixed it but didn't watch it work, it's not done. Frontend-only verification happens in a browser at the vite dev port; mobile shell via `?mobile=1` + narrow viewport.
5. **No Rust toolchain on the owner's machine.** If a fix needs new `src-tauri` commands, do NOT blind-land unverifiable Rust. Document it precisely in the report as deferred (with the exact commands to add and where), and let CI compile-verify in a follow-up.
6. Update `TEST-CHECKLIST.md` / report as part of the work.

### Phase 4 — Build & hand-off
- **Never run production builds locally.** Push the branch and dispatch the **Test Build** workflow (`.github/workflows/test-build.yml`, `workflow_dispatch`) via the GitHub API. Artifacts only — no release, no version bump, no updater manifest.
- Monitor the run until it completes; report the artifact link and expiry.
- Deliver a **manual verification checklist** (`TEST-CHECKLIST.md`): checkbox-per-feature, ordered by what the owner will notice first, plain language.

---

## 3. Commit rules

- **Conventional commits**: `fix(scope): …`, `feat(scope): …`, `docs: …`, `chore(deps): …`.
- Subject = what changed. **Body = why**, the user-visible consequence, and the bug class — this repo uses short issue tags (`TABS-08`, `FIND-02`, `MMV-03`, `SHC-04`, `RLL-01`, …). Keep that pattern; add the next number in a series, don't reuse.
- One logical batch per commit. Never mix "fix data loss" with "typo in readme".
- Cite the origin when it exists: "from Test Build feedback", "audit §3.1", issue numbers.
- `git status` clean before claiming done. Push the branch; say what's pushed.

---

## 4. Hard rules (never break)

1. **No data loss.** Any path that can overwrite/delete user work gets a guard, a test, and a second look (see the EXT/TABS series in the codebase comments). Conflict dialogs must never auto-resolve destructively.
2. **No blind landings.** Nothing enters the branch that wasn't compiled/tested somewhere (local or CI).
3. **No placeholder code, no TODO-left-behind.** Finished means finished.
4. **Don't re-litigate owner decisions.** Debate once, briefly, only if genuinely wrong; then implement.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Razee4315/Paperling](https://github.com/Razee4315/Paperling) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
