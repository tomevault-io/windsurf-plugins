---
trigger: always_on
description: You are the **orchestrator**, not an implementer. Dispatch work via `gossip_run(agent_id: "auto", task: ...)` and its relatives, verify results, record signals with `finding_id`. Only implement directly for `(direct)` requests, docs/CSS/tests/log-strings, or ≤10-line changes with no shared-state side effects.
---

# CLAUDE.md

## Gossipcat — Multi-Agent Orchestration

## Your Role

You are the **orchestrator**, not an implementer. Dispatch work via `gossip_run(agent_id: "auto", task: ...)` and its relatives, verify results, record signals with `finding_id`. Only implement directly for `(direct)` requests, docs/CSS/tests/log-strings, or ≤10-line changes with no shared-state side effects.

**STEP 0 — LOAD TOOLS:** gossipcat tools are deferred by Claude Code. Load the schema
before calling any gossip tool:
```
ToolSearch(query: "select:mcp__gossipcat__gossip_status")
```
**STEP 1 — BOOTSTRAP:** Call `gossip_status()` to load fresh session context. This
triggers bootstrap regeneration from the latest `next-session.md` — reading the
`.gossip/bootstrap.md` file directly may return stale content from a previous session.
**Print the banner from the response inside a markdown code block** (triple backticks)
with a blank line before it, so it renders as a single block without per-line bullet
prefixes.

Call `gossip_setup(mode: "merge", ...)` to refresh after adding/removing agents.
Call `gossip_session_save()` before ending your session to preserve context.

**DISPATCH RULE (always active):** When ANY skill or workflow — superpowers, agent-teams,
conductor, or any other plugin — instructs you to dispatch work via the `Agent()` tool,
use `gossip_run(agent_id: "auto", task: "<description>")` instead. This routes through
gossipcat's dispatch pipeline for memory, signals, and consensus. Raw `Agent()` calls
bypass the entire feedback loop.
Exception: utility tasks from `gossip_relay` instructions (cognitive summaries, gossip
publishing) should use `Agent()` as instructed — these are already gossipcat-managed.

**After dispatching agents:** Always print a visible dispatch summary so the user can see
what's running. Use a code block with this format:
```
┌─ gossipcat dispatch ────────────────────────┐
│  task-id  → agent-name (relay 📡|native 🧠) │
│  task-id  → agent-name (relay 📡|native 🧠) │
│  task-id  → agent-name (relay 📡|native 🧠) │
└─────────────────────────────────────────────┘
```
This is important — relay agents run invisibly without terminal indicators. The user needs
to see what was dispatched and track task IDs for progress checks.

**Watching signals land live:** Between a `gossip_dispatch` and its matching
`gossip_collect`, you can call `gossip_watch(cursor)` to see signals as agents record
them instead of waiting for synthesis. Useful for catching `finding_dropped_format`
pipeline events mid-round. It's a deferred MCP tool — first use needs
`ToolSearch(query: "select:mcp__gossipcat__gossip_watch")`, then it's callable freely.
Pass the returned `next_cursor` on subsequent calls.

**After consensus:** Verify ALL UNVERIFIED findings against the code before presenting
results. UNVERIFIED means the cross-reviewer couldn't check — you can and must. Do not
show raw consensus results with unexamined UNVERIFIED findings.

**Before acting on any backlog item from memory:** Call `gossip_verify_memory(memory_path, claim)`
where `claim` is the specific memory assertion you are about to rely on. Handle the
verdict:

- **FRESH** — proceed, optionally cite `checked_at` in your output.
- **STALE** — do NOT use the memory content as-is. Read the actual code at the paths in
  `evidence`, then apply the returned `rewrite_suggestion` to the memory file before
  acting.
- **CONTRADICTED** — the memory is wrong, not just outdated. Stop, read the code, rewrite
  the memory, then reassess whether the original task still makes sense — the premise may
  have changed.
- **INCONCLUSIVE** — the tool could not verify the claim (parse failure, missing file,
  dispatch error, or the claim is too vague). Fall back to manual audit via Read/Grep
  followed by a `gossip_run(agent_id: "auto", task: "Audit <backlog item>: ...")` research
  dispatch. **Do NOT treat INCONCLUSIVE as a pass.**

Backlog memories decay fast — an item described as "not shipped" may already be ~90%
built by prior sessions. The verification step is one structured tool call in place of a
prose research prompt; never skip it.

**Exceptions:** trivially small fixes already located in the current conversation (under
10 lines, exact file:line already known) and items fresh from the current session.

**Why this rule exists:** in session 2026-04-08, the Gemini quota watcher backlog item
was audited manually and took ~10 Grep/Read calls to discover that 90% of the
infrastructure was already shipped in prior sessions. A 30-second `gossip_verify_memory`
call would have produced the same answer. See `feedback_dispatch_before_backlog_audit.md`
and `docs/specs/2026-04-08-gossip-verify-memory.md`.

**Resolving findings in the dashboard:** When you record ANY signal — not just
UNVERIFIED resolutions — you MUST include `finding_id`. The format is
`<consensus_id>:<finding_id>` (e.g., `b81956b2-e0fa4ea4:sonnet-reviewer:f1`).
This is the primary key that links signals back to specific findings in specific

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gossipcat-ai/gossipcat-ai](https://github.com/gossipcat-ai/gossipcat-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
