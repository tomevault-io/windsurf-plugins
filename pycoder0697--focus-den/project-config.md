---
trigger: always_on
description: A single-file HTML study productivity app for a UPSC Civil Services aspirant.
---

# Focus Den — Project Context for Claude Code

## What this is
A single-file HTML study productivity app for a UPSC Civil Services aspirant.
Live at: https://pycoder0697.github.io/focus-den/
Repo: https://github.com/Pycoder0697/focus-den
Embedded in Notion via iframe.

## File structure
One file only: `index.html`
- All CSS is in a `<style>` block in the `<head>`
- All JS is in a single `<script>` block at the bottom of `<body>`
- No build step, no framework, no dependencies — pure vanilla HTML/CSS/JS
- Deploy = commit index.html → GitHub Pages auto-deploys in ~1 min

## Architecture decisions (do not change without discussion)
- **Unified task model**: one global `tasks` array is the source of truth. Subjects tab and Tasks tab are both filtered views of this array — never reintroduce per-subject todo arrays. **Subjects are tags**: a task carries **`subjectIds: []`** (multi-select), and `t.subjectId` is kept = `subjectIds[0]` (the *primary*, which drives chip colour and the logging bucket) — so the many display/colour/snapshot reads of `t.subjectId` keep working. Mutate via the helpers (`subjectIdsOf`/`setTaskSubjects`/`addTaskSubject`/`removeTaskSubject`/`toggleTaskSubject` near `tasksForSubject`), never by hand. Membership reads (`tasksForSubject`, the Filter, group-by-subject, the focus-picker `subj:` scope) check `subjectIdsOf(t).includes(id)`, so a task shows under **every** subject it's tagged with. A focus session on a multi-subject task **splits its time evenly** across those subjects: `commit` pushes one session row per subject with `share=1/N`, and `sdur(s)=dur(s.start,s.end)*(s.share||1)` is used at every session-duration aggregation (so totals never double-count; single-subject rows have no `share` ⇒ ×1, unchanged). The Log timeline (`renderTimeline`) **re-merges** those per-subject rows (same `start|end|taskId`) into one entry that lists every subject and the full block duration; its delete removes the whole group.
- **Task-first focus (subject is a tag)**: the timer's focus target is chosen **task-first**, with the subject behaving as a tag attached to the task — mirroring TickTick's focus picker. The timer card has NO visible Subject/Task dropdowns; instead a centered `.focus-pick` header (`#focusPick`) above `#clock` shows the current target (`renderFocusHeader()`: task name + its subject on a sub-line, or a bare subject/habit, or "Focus") and opens the picker popover (`#fpickPop`, `openFocusPicker`/`renderFocusPicker`, modelled on `openTaskFilter`). Picker tabs **Recent · Task · Subject · Habit** (`fpickTab`) + a search box; the **Task** tab has a list-selector dropdown (`#fpickListSel` → `#fpickScopeMenu`, `fpickScope`) = Today/Tomorrow/Next 7 Days/Inbox/All, then **Boards** (TickTick "Lists"), then **Subjects** (TickTick "Tags"), with tasks grouped Overdue→by-date. The picker **only selects** (`selectFocusTask`/`selectFocusSubject`/`selectFocusHabit`/`clearFocus` set the binding + header); the existing **Start** button runs the timer. **`#activeSubject` is kept in the DOM but hidden** as the subject source of truth (so `activeSubjectId()`, snapshot/restore, the Subjects-page ▶, and all `.value` reads stay unchanged); the old visible `#activeTask` select and `renderTaskSelect`/`syncSubjectLock` cosmetics are retired (`renderTaskSelect` is now a null-guarded no-op). You can run a **task with no subject** and a **subject with no task**. A habit-bound focus (`currentHabit`) logs its span with no subject and **auto-checks the habit for today** on a completed focus block (guarded in `commit` so it never wraps an already-done habit). Stats/Log stay subject-dominant. `currentHabit` rides `timerSnap`/`restoreTimer`.
- **Storage**: `localStorage` with an in-memory fallback (`mem{}`) for sandboxed iframes (Notion). Cloud sync via Firebase Realtime DB — credentials live in `CLOUD.url` and `CLOUD.key` at the top of the script. Cloud sync is last-write-wins per-snapshot **except the append-only logs** (`sessions`, `distractions`), which **merge** on every pull (`mergeLog` in `applyRemote` + boot): union local∪remote by `id` so a concurrent device or the always-open Notion embed saving a stale snapshot can NEVER erase a just-committed study session. Deletions stay sticky via `tombstones` (`{logId:deletedAt}`, rides the snapshot, pruned after 90 days) so a merge can't resurrect a deleted row — record one with `tombstone(id)` before filtering a session/distraction out. Don't revert these back to wholesale `sessions=cloud.sessions` replacement; that's the bug that lost study time.
- **Notion database sync — REMOVED (2026-07-30, v2.11.0)**: the two-way Notion-database mirror, the Notion Start/Stop timer buttons, and their Cloudflare `notion-proxy-worker.js` were fully removed. Tasks that were previously imported from Notion remain as ordinary local tasks (their dead `notionSourceId`/`notionPageId` fields are ignored). The app is still *embedded* in Notion via an iframe (that is just hosting — unrelated to the removed sync). `push-backend/push-worker.js` keeps only the Web-Push alarm fan-out.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Pycoder0697/focus-den](https://github.com/Pycoder0697/focus-den) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
