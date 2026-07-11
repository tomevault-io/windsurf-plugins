---
trigger: always_on
description: Open-source cooperative platform. Goal: end poverty, unite humanity.
---

# HumanityOS — Claude Context

Open-source cooperative platform. Goal: end poverty, unite humanity.
Live: https://united-humanity.us | GitHub: https://github.com/Shaostoul/Humanity
SSH alias: `humanity-vps` (server1.shaostoul.com)

> **⚠️ START HERE (mandatory, every TOP-LEVEL session):**
> 0. Run `just clean-worktrees` to kill stale AI context before it corrupts new work. **OPERATOR/ORCHESTRATOR ONLY — a dispatched subagent must NEVER run this.** The script (2026-07-01 hardened, `scripts/clean-worktrees.sh`) now refuses to touch a worktree with uncommitted changes or unmerged commits, but a subagent told to "read CLAUDE.md first" (a routine review-agent instruction) used to read this step literally and run it as its own first action, force-deleting sibling worktrees mid-review — it happened 3 times in one day before the script was hardened. If you are a subagent reading this file as background context for a specific delegated task, skip this step entirely; it is not part of your task.
> 1. **READ `docs/PRIORITIES.md`** — strict-ranked TACTICAL backlog. The TOP item of TIER 0 is what gets worked on next. New convention as of v0.283.x. Its STRATEGIC companion is **`docs/ROADMAP.md`** (themed, status-badged, the public roadmap AND the build to-do list; the website renders it from `data/roadmap.json` via `scripts/roadmap-to-json.js`). When you change strategic scope, update ROADMAP.md + regenerate the JSON.
> 2. **READ `data/coordination/orchestrator_state.json`** — running session journal. Tells you what the previous orchestrator was doing, what decisions were made, what scopes have active claims, what NOT to redo.
> 3. **Run `node scripts/agent-status.js`** — per-scope coordinator-friendly summary aggregating `data/coordination/sessions/*.json`.
> 4. Read `docs/FEATURES.md` for complete feature inventory with file paths (never rebuild what exists)
> 5. Read `docs/PAGES.md` for the canonical UI page registry (its heading carries the live native/web page counts; don't trust copies of the numbers elsewhere)
> 6. Read `docs/STATUS.md` for what's built vs planned (never re-plan completed work)
> 7. Read `docs/BUGS.md` for resolved bugs (never re-fix a fixed bug)
> 8. Read `docs/SOP.md` for version sync, deploy, and development procedures
> 9. Read `docs/design/ui-system.md` before touching any widget, page, or visual code
> 10. Read `docs/design/infinite-of-x.md` before writing any list-shaped literal in code
> 11. Read `docs/design/storage-architecture.md` before touching any storage / signed object / federation code
> 12. **Before pushing a release**: `git status --short` and stage any untracked .rs/.ron/.csv. Local builds pass with untracked files; CI fails on fresh checkout.
> 13. **After pushing a Rust release**: run `just build-game` to produce a versioned local exe — CI doesn't build Windows.
> 14. Before proposing ANY new feature, check FEATURES.md first. If it's listed, enhance it instead.
> 15. If agents report editing files under `native/src/`, `server/src/`, or `crates/`, those paths don't exist anymore. Run `just clean-worktrees` and redo against the real `src/` tree.
> 16. **Before claiming a multi-AI scope**, check `data/coordination/agent_registry.ron` for ownership rules and the `agent_sessions` SQLite table for active claims.
> 17. **Before ending the session** with significant changes: update `docs/PRIORITIES.md` (what's next) AND `data/coordination/orchestrator_state.json` (why we got here). End the reply with a "Next:" pointer (the session-end convention adopted in v0.283.x). If the journal has grown large (run `just brief`; or it's > ~150 KB), run `just rotate-journal` to archive old decisions to `docs/history/journal-archive-<month>.md` (keeps newest at the bottom).
> 18. **Before quoting algorithms / tech specifics in user-facing copy** (X posts, README, marketing): grep the actual code or read the Cryptography section. Memory + docs may lag behind code during migrations.
>
> **When things go wrong:** read `docs/INCIDENT-PLAYBOOK.md` (recipes for live failures + lessons from past incidents).
> **For long-term posture:** `docs/BUS-FACTOR.md` (succession), `docs/SECURITY-CADENCE.md` (mandatory periodic exercises), `docs/HEALTH-DASHBOARD.md` (SLOs + alert criteria).
>

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Shaostoul/Humanity](https://github.com/Shaostoul/Humanity) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->
