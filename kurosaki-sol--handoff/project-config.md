---
trigger: always_on
description: Three-phase session handoff for preserving reusable learnings, active project state, and durable project facts. Use when context is nearly full, the user is about to clear or close the session, says "handoff", "brain-dump", "save state", "bilan", or similar, or a meaningful work unit has just finished. Writes evidence-aware memories and HANDOFF.md without committing to git.
---


# Handoff — Session End Persistence

## Problem

Long agent sessions accumulate three kinds of value that die on `/clear`, `/compact`, or a fresh terminal:

1. **Reusable learnings** — non-obvious debugging techniques, workarounds, gotchas that help across *any* future project.
2. **Project state** — branches in flight, open PRs, blockers, TODOs. Painful to reconstruct from git log.
3. **Durable project facts** — architecture decisions, threat model clarifications, team preferences the codebase can't express.

On top of that, memories **rot**: a memory written 6 months ago may describe a reality that no longer exists. The worst failure isn't a long prompt; it's an agent confidently using stale info as if it were current. This skill tracks enough metadata per memory that future sessions can distinguish `fresh` from `suspect` from `stale` **on evidence**, not on calendar age.

## Trigger conditions

Invoke when ANY of these is true:

- Context window at ~85%+ and the session has produced meaningful work
- User types `handoff`, `brain-dump`, `save state`, `bilan`, `bilan complet`, `handoff complet`
- User mentions they're about to `/clear`, close the terminal, or step away
- A coherent work unit just finished (PR batch shipped, audit complete, feature merged)
- Proactively, at natural transition points in very long sessions (e.g. after 3+ hours)

Don't invoke for trivial sessions, or mid-task when the user just needs context compression.

## Workflow

### 0. Preamble — identify the scope

Before writing anything:

- **Working directory + git state** — `pwd`, `git rev-parse --show-toplevel 2>/dev/null` (no git → non-repo handling applies), `git rev-parse HEAD 2>/dev/null` for each relevant repo.
- **Existing agent bootstrap files** — read `AGENTS.md` and `CLAUDE.md` if present.
- **Existing HANDOFF.md** — read; augment, don't overwrite.
- **Existing memory dirs** — check agent-specific memory locations, especially `~/.claude/projects/<slug>/memory/` and `~/.codex/memories/`; read indexes only when present and relevant.
- **Session summary** — what was done, decided, shipped, blocked.

Decide which phases apply: Phase A if non-obvious learnings exist; Phase B always if in-flight work; Phase C if agent bootstrap files can make future resume safer without polluting a repo.

### Portability model — one canon, thin adapters

`HANDOFF.md` is the canonical artifact. It must be enough for a cold human, Claude Code, Codex CLI, OpenCode, or another compatible agent to resume without relying on hidden context.

Agent-specific files are adapters:

- **Codex CLI** — prefer `AGENTS.md` as the reliable project bootstrap. It should point the next Codex session at `HANDOFF.md` and list only durable rules. Do not assume Codex will auto-load a prior chat after `/clear`.
- **Claude Code** — use `CLAUDE.md` when the repo already owns one, plus Claude's project memory directory when useful.
- **Other agents** — keep them on the `HANDOFF.md` path unless their documented bootstrap file is present.

Never maintain separate competing handoff narratives. If an adapter disagrees with `HANDOFF.md`, update the adapter to point back to the canon.

---

## Phase A — Extract reusable learnings to persistent memory

For each candidate learning, pass the quality gates:

- [ ] **Reusable** across future sessions — not a one-off bug fix
- [ ] **Non-obvious** — required discovery, not a docs lookup
- [ ] **Specific** — has concrete triggers, symptoms, file paths
- [ ] **Verified** — confirmed during the session
- [ ] **Not already in memory** — check `MEMORY.md` first; update in place if it exists

Persistent memory is agent-specific. Write only memories that pass the gates, and avoid pretending every agent has the same recall behavior:

- **Claude Code**: files go in `~/.claude/projects/<project-slug>/memory/`. Two steps per entry: write the file, add an index pointer to `MEMORY.md`.
- **Codex CLI**: if Codex memory is available in the environment, mirror only durable, compact memories into `~/.codex/memories/` using the same schema. Still write `HANDOFF.md` and `AGENTS.md`; they are the reliable resume path after `/clear`.
- **Unknown agent**: skip hidden memory unless the user or environment documents a location. Put resume-critical facts in `HANDOFF.md`.

### A.1 — The memory schema (v2)

```yaml
---
name: {short title — what it is}
description: {one-line — specific enough to judge relevance from index alone}
type: feedback | project | reference | user
volatility: none | low | code-anchored | high
state: fresh
created_at: YYYY-MM-DD
last_verified_at: YYYY-MM-DD
pin:
  type: git-commit | git-path | git-multi | file-hash | ttl | user-override-only | semantic
  {type-specific fields — see A.2}
anchors:
  - path: src/models/FaucetClaim.ts
    sha256: <hash of file content at write-time>
    claim_excerpt: "FaucetClaim has no userId field on stealth path"
domain_tags: [faucet, privacy, mongoose-schema]

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kurosaki-sol/Handoff](https://github.com/kurosaki-sol/Handoff) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
