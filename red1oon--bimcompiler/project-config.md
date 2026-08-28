---
trigger: always_on
description: **EXTRACT OR COMPILE ONLY.** Query the database. Copy patterns you find. Compute positions via verbs. Never invent.
---

# BIM Intent Compiler — Session Protocol

## PRIME RULE
**EXTRACT OR COMPILE ONLY.** Query the database. Copy patterns you find. Compute positions via verbs. Never invent.

**NEVER TOUCH PRODUCTION.** `deploy/live/` is the production snapshot — do not edit directly. All dev work goes to `deploy/dev/` ONLY. Read `deploy/OCI_UPLOAD.md` §RULES before any OCI upload.

## ⏸ PUSH PAUSE — LIFTED 2026-07-17
Stood 2026-07-11→2026-07-17 (local/localhost-only verification while a large merge burst got
batch-verified). **User directive 2026-07-17: "now push permission is ON, ie always push to live
until user decides to pause."** Push freely (normal fast-forward pushes / PRs — not force-push, not
skipping the localhost/§-witness verification habit this project still runs on) until the user says
to pause again. If they do, the same discipline as before applies: commit locally, verify on
localhost, no push/PR, and every Agent-tool dispatch must carry that instruction explicitly since a
fresh worker has no memory of it.

## WORK-TO-ZERO (the backlog contract — enforced every session)
**No standing backlog file right now — `prompts/archive/FRONTEND_LANE_MASTER.md §NEW BACKLOG` (archived 2026-07-11, prompts-audit) DRAINED 2026-07-08 (every
top-level item `✅`; same retirement treatment as the earlier `§OUTSTANDING` band, RETIRED 2026-06-20 → archived
to `prompts/archive/FRONTEND_LANE_MASTER_OUTSTANDING_drained_2026-06-20.md`). Do NOT re-walk either band, and
do NOT re-derive "is it still stale" by re-reading that 523-line file — it's settled. Two small real items were
found buried in its already-✅ prose (not lost, just not worth their own file): (1) bim-ootb `runSave()`'s 57s
wall-clock never profiled to WHERE the time goes (message-clarity was fixed, the budget wasn't); (2) bim-ootb
`38-offline-pwa.spec.js`'s hardcoded `VIEWER_URL` path blocks CI-wiring 3 offline/PWA specs (fix named in
`GH_DEPLOY_ISSUES.md` Issue 4). Pick either up directly if ever prioritized — no rediscovery needed.**

**The RULE stays live even with no active list** — the next time a dictated multi-item job is given, work it
top-to-bottom to zero in whatever file it lives:
- take the top open item → spec → implement → witness/§-log → mark it `✅ DONE (witness)` in the list → next item.
- **Do NOT stop and report "it's parked."** Keep going through the list. The default is *continue*, not *hand back*.
- **Stop only when:** (a) the user interrupts (their call, any time), or (b) an item genuinely needs a user
  fact/decision you cannot EXTRACT — then mark it `⛔ BLOCKED: <the one question>` and **move to the next item**
  (never loop on it, never silently drop it).
- **Session end** = every item is `✅` or `⛔`. Report only the ✅ list + the ⛔ questions. If the list isn't zero
  and you weren't interrupted, you stopped too early — that is the failure this rule exists to kill.
- Shared working tree: editing `~/bim-ootb/` is now **BLOCKED by a PreToolUse hook** (verified 2026-06-06) —
  work in a `/tmp/wt-*` worktree, never the shared checkout. See `~/.claude/hooks/block-shared-tree.sh`.
  **⚠ `bim-compiler` has NO equivalent hook** — this checkout (`/home/red1/bim-compiler`) is fully editable
  by any concurrent session. Confirmed live 2026-07-11: two parallel sessions both directly edited
  `build/room_walker.js`/`scripts/compile_rooms.py` in this shared tree at the same time — non-destructive
  only by luck (verified after the fact, nothing lost). Prefer a `/tmp/wt-*` worktree here too when a task
  might overlap with concurrent work on the same files; if editing the shared tree directly, expect this
  risk and verify post-hoc (diff/syntax-check) rather than assume no collision occurred.
- **Concurrent branches (N-terminal workflow):** with multiple terminals, `main` advances under you.
  - A PR showing **`BEHIND`/`DIRTY` is *sync*, NOT a redo** — `git fetch origin && git merge origin/main`,
    re-run witnesses, push. Your commits are preserved (you layer main's in).
  - Let **auto-merge** keep it current (`gh pr merge <n> --auto --squash`; the github-actions bot also enables it)
    — but **verify it actually landed**: a squash-merge + a late push *orphans* the new commit (observed PR #138,
    2026-06-05). After a branch is squash-merged, start the follow-up off **fresh `origin/main`** — never re-use it
    (its history collides with the squash → `DIRTY`).
  - **`sw.js` is the conflict magnet** (every deploy bumps `CACHE_VERSION` + `PRECACHE_ASSETS`). On conflict: KEEP
    BOTH precache additions, take the HIGHER `CACHE_VERSION`. Never resolve by dropping the other session's hunk.
  - The worktree isolates working-dir + checked-out branch, **NOT** line-level merge conflicts on shared files.
- **Docs deploy (READ `prompts/DOCS_DEPLOY_POLICY.md` + `prompts/DOCS_DEPLOY_GUARD.md`):** publish docs ONLY
  via **`scripts/safe_gh_deploy.sh`** (the no-shrink seatbelt — W-DEPLOY-GUARD) from your FULL working branch.
  **NEVER run bare `mkdocs gh-deploy`** — it overwrites the whole `gh-pages` site and a stale/thin tree silently
  wipes live pages (happened twice). The seatbelt aborts SOFT (exit 1, `gh-pages` untouched) if a publish would

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [red1oon/BIMCompiler](https://github.com/red1oon/BIMCompiler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
