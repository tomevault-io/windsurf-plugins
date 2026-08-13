---
trigger: always_on
description: React game client for SideQuest. Connects to the Python `sidequest-server`
---

# CLAUDE.md — SideQuest UI (React/TypeScript)

React game client for SideQuest. Connects to the Python `sidequest-server`
WebSocket at `ws://localhost:8765/ws` (per ADR-038, post-port).

## CRITICAL: Personal Project

This is a personal project under the `slabgorb-org` GitHub organization.
- **No Jira integration.** Never create, reference, or interact with Jira tickets.
- **No 1898 org.** Nothing goes to the work GitHub org. Ever.
- All live repos are under `github.com/slabgorb-org/` (use `gh ... -R slabgorb-org/<repo>`). The historical Rust prototype `sidequest-api` remains under `github.com/slabgorb/`.

## SideQuest System Overview

Six repos compose the SideQuest stack (Python backend per ADR-082, ported from the Rust prototype 2026-04):
- **sidequest-server** — Python/FastAPI game engine and WebSocket API on port 8765
- **sidequest-ui** — React/TypeScript game client (Vite, port 5173)
- **sidequest-daemon** — Python media services (image gen, music gen)
- **sidequest-content** — Genre packs (YAML configs, audio, images, world data)
- **sidequest-composer** — Standalone CLI: public-domain notation → rights-free audio (offline)
- **sidequest-understudy** — Naive simulated-player playtest client

Orchestrator repo (`orc-quest`, also cloned as `oq-1` / `oq-2`) coordinates sprint tracking, docs, ADRs, and cross-repo scripts.

## Quality Rules

- No stubs, no hacks, no "we'll fix it later" shortcuts
- No skipping tests to save time
- No half-wired features — connect the full pipeline or don't start
- If something needs 5 connections, make 5 connections. Don't ship 3 and call it done.
- **Never say "the right fix is X" and then do Y.** Do X.
- **Never downgrade to a "quick fix" because you think the context is "just a playtest."**
  Every playtest is production tomorrow. Fix it right.

## Development Principles

### No Silent Fallbacks
If something isn't where it should be, fail loudly. Never silently try an alternative
path, config, or default. Silent fallbacks mask configuration problems and lead to
hours of debugging "why isn't this quite right."

### No Stubbing
Don't create stub implementations, placeholder modules, or skeleton code. If a feature
isn't being implemented now, don't leave empty shells for it. Dead code is worse than
no code.

### Don't Reinvent — Wire Up What Exists
Before building anything new, check if the infrastructure already exists in the codebase.
Many systems are fully implemented but not wired into the server or UI. The fix is
integration, not reimplementation.

### Verify Wiring, Not Just Existence
When checking that something works, verify it's actually connected end-to-end. Tests
passing and files existing means nothing if the component isn't imported, the hook isn't
called, or the endpoint isn't hit in production code. Check that new code has non-test
consumers.

### Every Test Suite Needs a Wiring Test
Unit tests prove a component works in isolation. That's not enough. Every set of tests
must include at least one integration test that verifies the component is wired into the
system — imported, called, and reachable from production code paths.

### Backend Language
The server (`sidequest-server`) is Python/FastAPI per ADR-082, ported from a
Rust prototype in 2026-04. The Rust codebase is preserved read-only at
https://github.com/slabgorb/sidequest-api for historical reference; older ADRs
that show Rust code are historical illustration only — see `docs/adr/README.md`
for the translation table. New backend code goes in Python. Media services
(`sidequest-daemon`) remain Python for inference library maturity (Z-Image /
ACE-Step). The narrator LLM path uses the Anthropic Python SDK by
default per ADR-101 (supersedes ADR-001; `claude -p`/Ollama are opt-in
non-default backends). (Kokoro TTS was formerly in this list; TTS has been
removed from the system.)

## OTEL Observability Principle

Every backend fix that touches a subsystem MUST add OTEL watcher events so the GM panel
can verify the fix is working. Claude is excellent at "winging it" — writing convincing
narration with zero mechanical backing. The only way to catch this is OTEL logging on
every subsystem decision:

- **Intent classification** — what was the action classified as, and why?
- **Agent routing** — which agent handled the action?
- **State patches** — what changed in game state (HP, location, inventory)?
- **Inventory mutations** — items added/removed, with source
- **NPC registry** — NPCs detected, names assigned, collisions prevented
- **Trope engine** — tick results, keyword matches, activations
- **Encounter engine** — beat selections, metric changes, resolution

The GM panel is the lie detector. If a subsystem isn't emitting OTEL spans, you can't
tell whether it's engaged or whether Claude is just improvising.

**Not needed for:** Cosmetic UI changes (labels, spacing, colors).

## Architecture Decision Index

ADRs live in the orchestrator repo at `orc-quest/docs/adr/`. See
`orc-quest/docs/adr/README.md` for the canonical index. Before designing
or modifying a subsystem, check the relevant ADR:

Particularly relevant to this UI repo:

| Domain | ADRs |
|--------|------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [slabgorb-org/sidequest-ui](https://github.com/slabgorb-org/sidequest-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
