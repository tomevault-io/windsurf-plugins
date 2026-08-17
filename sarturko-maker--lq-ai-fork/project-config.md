---
trigger: always_on
description: Fork of [LegalQuants/lq-ai](https://github.com/LegalQuants/lq-ai) (Apache-2.0), baseline `f91149a` (post-v0.4.0). See `UPSTREAM.md`.
---

# CLAUDE.md — LQ.AI Fork

Fork of [LegalQuants/lq-ai](https://github.com/LegalQuants/lq-ai) (Apache-2.0), baseline `f91149a` (post-v0.4.0). See `UPSTREAM.md`.

## The goal (why we forked)

**This fork is for IN-HOUSE legal teams — a company's own legal department — NOT law firms.** The org
is one company; its users are that company's lawyers. Say "the company's legal team", never "the firm".

Upstream organises the UX around tools (Skills, Playbooks, Tabular Review — 11 flat tabs). We organise it
around an in-house lawyer's **unit of work**, inside configurable **practice areas** (Commercial, Disputes,
M&A, Privacy, Employment…). Concretely:

- Each practice area is one **Deep Agent** (LangChain `deepagents`) that picks its own tools, skills,
  playbooks and MCP servers, and fans out subagents when useful. No fixed pipelines.
- Each practice area defines a configurable **unit of work** ("Matter" in Commercial, "Programme — GDPR"
  in Privacy) that loads the area's tools/skills/playbooks/MCPs and accumulates memory.
- **Memory accumulates at 4 levels**: company/client profile → practice area → user → unit of work.
  All four inject as context; the agent reads bulk material on demand.
- UX bar: effortless, like Claude Code. The user states intent; the agent works visibly
  (streamed tool calls, subagent fan-out, honest receipts).

Upstream's three "agentic" executors (playbooks, tabular, autonomous) are linear LangGraph pipelines —
Python walks the graph, the model fills JSON slots, no model-chosen tool call exists anywhere. We replace
the orchestration; we keep the substrate (gateway, brakes, audit, citations, skills format).

## Authoring boundary (maintainer ruling, 2026-07-08 — binding on Workstream B)

Two tiers, never blurred:

- **Personal tier — any user.** Users author skills and upload knowledge for THEIR OWN account and
  invoke them in plain chat (`/skill-name`). Personal modules NEVER reach a Deep Agent.
- **Org tier — admin only.** Deep Agents cut across the organisation, so only the ADMIN assembles
  them: which modules, which practice areas (admins create any area they want; shipped default
  templates exist for common ones). Users cannot configure, extend, or self-serve onto a Deep
  Agent — they ASK the admin. The only path from user-authored content to an org Deep Agent is
  propose → admin approve → Library → area binding (ADR-F067) — no other path, ever.
- **MCP is double-gated.** Users can NEVER connect an external MCP server (no user-level MCP at
  all), and the admin-level MCP module kind is itself still future work behind its own
  approval-gated milestone (ADRs 0014/0015).

## State of the pivot (keep current — stale info here is worse than none)

- F0-S1 done: langgraph 1.x + `deepagents==0.6.8` substrate landed; first model-driven tool loop
  proven live through the gateway (MiniMax-M3). Upstream code remains LEGACY: bugfix only — no new
  features, no refactors, unless the task IS the migration.
- ADR-F001..F005 are `accepted` — read them before structural work. Current slice: see
  `docs/fork/HANDOFF.md`.

## Architecture rules

- IMPORTANT: new agent code uses `deepagents` (pinned exact version) on langgraph 1.x. Do not extend
  the legacy executors (`api/app/autonomous/`, `api/app/playbooks/executor.py`, `api/app/tabular/`).
- KEEP unchanged: the Inference Gateway as the only egress and only key-holder; the `guarded_tool_call`
  chokepoint pattern (R4 cost cap / R5 halt / R6 grants) for every agent action; the audit contract
  (counts/types/IDs, never raw values); the Citation Engine; the SKILL.md format.
- Every LLM call routes through the gateway. Never add a direct provider call.
- "System proposes, user owns" (ADR-0013 D4) holds for the company/practice (read-only to agents) and
  user/autonomous memory tiers. The **unit-of-work (matter/programme) tier is auto-write-then-correct**
  (ADR-F042): the agent maintains it automatically; the human owns it *after* the write (correct/undo/delete,
  human-pinned corrections win). Pinning is an authenticated human action, never an agent tool.
- Transparency is load-bearing: every prompt, skill, agent instruction and tool grant must be
  readable in the UI or the source.

## Code rules — dependency injection & security

- Inject dependencies; never reach for globals. FastAPI `Depends` at the API edge; constructor
  arguments everywhere else (DB session, gateway client, skill registry, memory backends, agent
  instances). Wire-up happens once in the lifespan/composition root — no import-time I/O, no new
  module-level singletons. Tests substitute fakes through the same seams; don't monkeypatch what
  you can inject. Upstream's `app.state` + dependencies pattern is the exemplar — match it.
- Provider keys exist only inside the gateway — never in `api/`, `web/`, logs, or error messages.
- Authz on every endpoint; cross-user access returns 404, never 403 (no existence leaks).
- SQL is parameterized via SQLAlchemy; string-built SQL never passes review.
- Validate all input at the boundary with Pydantic; reject, don't sanitize.
- Treat retrieved documents, KB chunks, and shared memory as untrusted model input (prompt

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sarturko-maker/lq-ai-fork](https://github.com/sarturko-maker/lq-ai-fork) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
