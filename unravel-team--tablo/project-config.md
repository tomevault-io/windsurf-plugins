---
trigger: always_on
description: **Tablo** is a small floating animated cat that watches your Claude Code agents. It lives unobtrusively on screen and reflects activity through animation: when agents are running, Tablo runs and shows the agent count; when a session's context window enters the danger zone (>60%), Tablo becomes alarmed. **Tapping Tablo opens a panel** with the live context-window meter. This avatar-plus-panel model replaces an always-on-top window, which would take up too much screen space.
---

# Tablo — Implementation Plan

**Tablo** is a small floating animated cat that watches your Claude Code agents. It lives unobtrusively on screen and reflects activity through animation: when agents are running, Tablo runs and shows the agent count; when a session's context window enters the danger zone (>60%), Tablo becomes alarmed. **Tapping Tablo opens a panel** with the live context-window meter. This avatar-plus-panel model replaces an always-on-top window, which would take up too much screen space.

This document is the build plan for Claude Code to follow. Phase 1 ships Tablo (the avatar) and the context meter only; later phases add permissions, plan usage, and a dashboard.

---

## Project overview

**Goal (Phase 1):** A tiny always-on-top animated cat avatar that watches Claude Code session transcripts and reflects activity through its animation state. Tapping it opens a panel showing each active session's context-window fill as a live percentage bar, with a danger warning above 60%.

**Interaction model — two surfaces:**
- **Avatar** — a tiny (~64×64), transparent, always-on-top, click-through-except-on-the-cat window. Always visible. Animation is driven by live data (idle / running with agent count / alarmed).
- **Panel** — a second, larger window that toggles open near the avatar when the cat is tapped, and dismisses on tap-away or a second tap. Contains the context meter(s). Not always visible.

**Stack:**
- **Tauri 2** (Rust backend + webview frontend) — chosen for small footprint, native always-on-top, transparency, and multi-window support.
- **Frontend:** plain HTML/CSS/JS or a lightweight framework (see Open Questions). Keep it minimal for Phase 1.
- **File watching:** Rust-side, using the `notify` crate to tail JSONL transcripts.
- **State bridge:** Tauri events (`emit`/`listen`) to push updates from Rust to both windows.
- **Avatar art:** sprite-sheet or SVG/CSS animation (see Open Questions) — designed separately; the plan leaves an asset slot.

**Non-goals for Phase 1:** permission approval, plan/quota usage, dashboard, multi-machine/LAN. These are scoped in later phases so the architecture leaves room for them but does not build them yet.

---

## Data model reference

Claude Code writes one JSONL transcript per session:

```
~/.claude/projects/<project-path-slug>/<session-id>.jsonl
```

Each line is a JSON event. The relevant lines are `assistant` messages, which carry a `usage` object:

```json
{
  "type": "assistant",
  "message": {
    "model": "claude-...",
    "usage": {
      "input_tokens": 4521,
      "cache_creation_input_tokens": 18023,
      "cache_read_input_tokens": 112400,
      "output_tokens": 843
    }
  }
}
```

**Context occupancy** is computed from the **latest** assistant message (not a sum across turns):

```
context_used = input_tokens
             + cache_creation_input_tokens
             + cache_read_input_tokens
             + output_tokens

context_pct  = context_used / context_limit
```

`context_limit` is model-dependent (200,000 standard; 1,000,000 if the extended-window beta is active). Read `message.model` to pick the denominator.

**Compaction note:** Claude Code auto-compacts near ~95%, which drops effective context. Expect the value to climb then snap down — handle this so it does not look like a bug.

---

## Avatar state model

The cat's animation is a pure function of the aggregate live state. Define these states explicitly so animation and data stay decoupled:

| State | Trigger | Animation |
|-------|---------|-----------|
| **idle** | no active session | cat sits / sleeps |
| **running** | ≥1 active agent | cat runs; agent count rendered on/beside it |
| **alarmed** | any active session's context > 60% | cat agitated (still shows count) |

Precedence: **alarmed** overrides **running** overrides **idle**. The agent count is displayed in both running and alarmed states.

**"Agent count" definition (resolved):** the count is the number of active **sessions** — distinct recently-modified transcripts under `~/.claude/projects/`. This falls directly out of the active-session detection in Step 1.2, so no extra tracking is needed. Subagents are deliberately **not** counted here — a 7-way fan-out is still one session you're watching — but they are surfaced as a per-session detail (see Subagents below).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [unravel-team/tablo](https://github.com/unravel-team/tablo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
