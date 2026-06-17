---
trigger: always_on
description: Writing style, project structure conventions, session context and memory handling, and git workflow including protected branches and worktree-per-feature.
---


## Writing Style

Never use em dashes (--). Use a regular hyphen (-) instead in all generated text, copy, comments, documentation, and commit messages.

## Project Structure Convention

`AGENTS.md` is the canonical project-instructions file across Claude Code, Codex, Cursor, and other tools. Claude Code reads it via a one-line `CLAUDE.md` containing `@AGENTS.md`. Always structure projects with a lean root `AGENTS.md` and deeper context in subdirectory `AGENTS.md` files co-located with the code they describe.

- **Root `AGENTS.md`** - one-paragraph summary, resolved architecture decisions, cross-cutting conventions, repo structure map. Keep it under ~40 lines. This limit applies to project root AGENTS.md files. The global `~/.claude/CLAUDE.md` is exempt.
- **Subdirectory `AGENTS.md`** (e.g. `backend/AGENTS.md`, `contracts/AGENTS.md`) - loaded only when working in that directory. Can be as detailed as needed without polluting other contexts.
- **`.claude/settings.json`** - project-scoped MCP servers and shared config (safe to commit).
- **`.claude/settings.local.json`** - secrets and local env values (always gitignored).

When starting a new project, run `/init-project` to scaffold this structure automatically.

## Session Context and Memory

**Session startup:** Read `.agentic/context.md` as the first action of every session - standalone, never in parallel with other tool calls.

**Meta-divergence sweep at session start.** After reading `.agentic/context.md`, the conductor sweeps `.agentic/events.jsonl` for `meta_review_complete` events whose `original_task_id` is not present in `.agentic/.meta-divergence-surfaced`. For each such event with non-empty `data.divergence.critical_missed` or `data.divergence.major_missed`, emit at the next user-facing turn boundary:

```
META-DIVERGENCE: meta-Skeptic identified [Critical|Major] '<finding-title>' that original Skeptic missed on <task_id>. Original sign-off stands; review recommended before merging.
[phase: meta-divergence-critical]
```

Then append `original_task_id` to the tracker file. The sweep is a standalone scan - not parallel with other startup tool calls. Tracker file format is one `original_task_id` per line, append-only, gitignored under the `.agentic/` umbrella. File-absent equals empty set. This catches divergences whose meta-Skeptic completed asynchronously after the originating session ended.

**Pagination (vicious loop defense):** The sweep MUST NOT read the full `.agentic/events.jsonl` on every boot. It reads only events with `ts` strictly greater than the timestamp stored in `.agentic/.meta-divergence-last-sweep` (ISO8601 UTC, single line, file-absent = first run). On first run (no tracker file), the scan is capped to the most recent 100 lines of the events file. After the sweep completes, the conductor writes the current ISO8601 UTC timestamp to the tracker file (atomic: tmp + `mv`). This prevents the vicious loop where growing telemetry consumes ever more context on every session start. See `content/references/skeptic-protocol.md` Section 14 "Session-start sweep pagination" for the full procedure.

**Session context** is auto-written by the Stop hook to `.agentic/context.md` after every agent turn. (Legacy fallback: `~/.claude/projects/[hash]/context.md` - used only when `.agentic/context.md` does not exist.) `/wrap` is available for richer on-demand summarization. Update `MEMORY.md` (root `<cwd>/MEMORY.md`) at the end of any session where stable facts were learned. Close the session cleanly so the Stop hook can finish writing `context.md`: in the terminal CLI, use `/exit` rather than ctrl+c; in the desktop or web app, just close the window or tab normally rather than force-quitting.

**Knowledge-file routing (three distinct stores):**
- `<cwd>/MEMORY.md` - canonical durable facts; committed; auto-injected by Claude Code; written by `/wrap`, wrap-ticket, `/memory-update`.
- `.agentic/memory.md` - `/wrap`-internal rolling scratch only; gitignored; NOT auto-injected; NOT the same as root `MEMORY.md`.
- `.agentic/learnings.md` - structured fix-pattern learnings; committed; written by `learning-extractor` (mechanically) and `learnings-agent` (discretionary).

**Per-developer session log:** `.agentic/session-log/<developer_id>.jsonl` - per-developer session rollup written by the Stop hook. Committed to git via the `.agentic/session-log/` carve-out in `.gitignore` when `commit_telemetry: true` (default) and identity is confirmed; the commit happens at `/implement-ticket` Phase 8 as a SEPARATE commit on the PR branch. Teammates receive it on pull after squash merge. See `content/references/events-log.md` "Per-developer session log". Aggregated via `agentic-cost team`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Space-Dinosaurs/DinoStack](https://github.com/Space-Dinosaurs/DinoStack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
