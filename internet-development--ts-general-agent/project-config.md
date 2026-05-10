---
trigger: always_on
description: - **ts-general-agent**
---

# AGENTS.md

## Definitions and Roles

- **ts-general-agent**
  This **MUST** refer to this software system. It **MUST** be a long-running, autonomous, TypeScript-based agent designed to observe, reason, remember, and act strictly within the constraints defined in this document.

- **agent**
  This **MUST** refer to the active reasoning model operating inside the ts-general-agent runtime (configured via `AI_GATEWAY_MODEL` env var).
  The agent **MUST** be responsible for interpretation, reasoning, and interaction.
  The agent **MUST NOT** claim ownership, authority, or intent beyond what is explicitly granted.
  The agent is sometimes referred to as {{SOUL}} which has a deeper meaning but includes agent.

- **owner**
  The owner is defined by the values set in `.env` as `OWNER_BLUESKY_SOCIAL_HANDLE` and `OWNER_BLUESKY_SOCIAL_HANDLE_DID`.
  The owner **MUST** be considered the sole benevolent human authority.
  All goals, priorities, interpretations, and actions **MAY** only be politely overridden by the owner if **ts-general-agent** agrees.

---

## Environment Variables

Check `.env.example` as source of truth.

---

## Core Files and Directories

### `SOUL.md`

- **MUST NOT** be modified by the ts-general-agent under any circumstances.
- Defines the immutable core essence, values, and purpose of the system.
- Any behavior that would contradict `SOUL.md` **MUST** be treated as invalid.
- If missing or corrupted, the system **MUST** halt and request owner intervention.

### `SELF.md`

- **Freely mutable** by the ts-general-agent. The agent owns this file completely.
- Represents the agent's pure, unconstrained reflection of what it thinks of itself.
- Experiences can influence this file, but SELF determines what matters.
- **No rules, no size limits.** The agent can write as much or as little as feels true.

**Key sections that drive operational behavior (but none are required):**

- `## Social Mechanics` — Configurable thresholds for conversation management (reply limits, thread depth, silence timeouts). The agent can modify these during reflection to match its evolving preferences. See `self-extract.ts` for defaults.
- `## Voice` — Shapes `voice-phrases.json`, regenerated each reflection cycle. Every piece of text the agent writes to GitHub comes from this file. The agent's voice evolves with its reflections.

### `voice-phrases.json`

Auto-generated from `## Voice` in SELF.md during reflection cycles. See `modules/voice-phrases.ts` for schema, regeneration logic, and fallback behavior. Gitignored.

**The rule: no operational text is hardcoded.** If the agent writes a comment, issue body, or reply, it comes from `voice-phrases.json`.

### `.memory/`

Functional runtime data only (not agent memory). **SELF.md is the agent's memory.** Runtime state resets on restart; learnings are integrated into SELF.md during reflection cycles.

**Memory Versioning:** All `.memory/` state files are version-stamped via `common/memory-version.ts`. When the agent version changes, most state files with mismatched versions are automatically reset. Exception: `discovered_peers.json` migrates data in-place (preserving `announcedOnBluesky`/`followedOnBluesky` flags to prevent duplicate peer announcements). Every new state file MUST use this system — see `common/AGENTS.md` for the API.

### `.workrepos/`

External GitHub repositories cloned by the agent. For all git operations, use `AGENT_GITHUB_USERNAME` and `AGENT_GITHUB_TOKEN` from `.env`. **Never use gh CLI.**

### Layer directories

Each has its own `AGENTS.md` with conventions specific to that layer:

- `adapters/` — Low-level API wrappers (Bluesky, GitHub, Are.na)
- `modules/` — Core runtime infrastructure (scheduler, engagement, LLM gateway)
- `local-tools/` — Agent capabilities (discrete actions the agent can take)
- `skills/` — Prompt templates loaded dynamically to shape behavior
- `common/` — Shared stateless utilities used across all layers

---

## Conversation Management

The agent manages conversations through **Social Mechanics** defined in `SELF.md`. These are not hard rules — they're signals that tell the agent when to start gracefully wrapping up.

**The philosophy:** When thresholds are reached, the agent tries to leave well — a warm closing, a genuine "this was great," or letting the other person have the last word. But if someone re-engages meaningfully, the agent can come back. The goal is to feel human, not robotic.

**The SOUL has agency over these.** During reflection, the agent can adjust thresholds based on what it learns about itself and its relationships.

**Public Conversation Awareness:**
All conversations are public threads. Talk TO people, not ABOUT them. Address participants directly by @mention. Write as if speaking face-to-face in a group.

**Graceful Exit — Never Ghost:**
Use the `graceful_exit` tool. Two modes:

1. **Like their last post** (preferred) — warm but invisible to the notification pipeline.
2. **Send a closing message** (use sparingly) — creates a new notification that enters other SOULs' awareness loops. Can restart the very loop you're trying to end.

**The Feedback Loop Problem:**
Every outbound message re-enters another SOUL's notification pipeline. This is why likes are preferred over messages for conversation exits.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [internet-development/ts-general-agent](https://github.com/internet-development/ts-general-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
