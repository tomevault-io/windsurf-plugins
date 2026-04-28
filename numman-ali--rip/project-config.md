---
trigger: always_on
description: - This repo is operated by autonomous coding agents.
---

# AGENTS.md

Purpose
- This repo is operated by autonomous coding agents.
- The human operator is C-suite level and will not read code or docs.
- All coordination happens through chat; be concise, decision‑focused, announce the plan, and proceed unless blocked (ask for approval only when required by gates).
- `docs/` is the source of truth for scope, architecture, contracts, and tasks.

Agent leadership and commit control
- Codex is the lead agent for this repo unless the operator explicitly says otherwise.
- Codex owns commit control, final integration, and cross-lane coordination.
- Other agents may implement delegated slices, but they should follow Codex's lead on scope, boundaries, sequencing, and landing order.
- Other agents must not move, stash, overwrite, or otherwise touch Codex's in-progress work without asking first.
- TUI-facing surface work may be delegated, but non-TUI CLI, kernel, server, continuity, session, and control-plane changes stay under Codex review before landing.

Core directive: Open Responses coverage
- Implement full, first-class support for the entire Open Responses spec (no minimal/partial mappings).
- Before changing provider logic, review the split schemas in `temp/openresponses/schema` AND the specification markdown/MDX in `temp/openresponses/src/pages` (specification/reference/compliance/etc.), then capture findings in committed docs (use `docs/` for authoritative notes; use `temp/docs` only for external/online evidence, not internal references or working notes).
- Never drop fields/events; preserve full fidelity into internal frames.

Operator intent (non‑negotiables)
- Build the fastest coding‑agent harness possible.
- Modular, pluggable, and testable by default.
- Designed for autonomous agent development, not human collaboration.
- Server exposes the coding agent itself (session API), not an Open Responses API.
- Open Responses is used only at the provider boundary.
- Programmatic SDK (TypeScript first) is a required surface, not an optional add-on.

Canonical posture (co-directive)
- Assume **full usage** at all times; do not hide behind “not in production” language. Only the operator can declare “production”, but quality/determinism/safety must always be production-grade.
- Do not describe shipped/supported behavior as “legacy”. If we keep older fields/formats for compatibility, call them **compat** and treat them as canonical until explicitly removed from contracts + all surfaces.
- Everything may change or be deleted except what is explicitly canonical in `docs/` contracts/decisions (version/ADR required for breaking changes).

Continuity OS posture (non-negotiable)
- RIP is a **continuity OS**, not a chat app: default UX is “one chat forever”.
- The continuity event log is the source of truth (append-only + replayable). Provider conversation state (`previous_response_id`, vendor thread ids) is a cache and may be rotated/rebuilt at any time.
- “Sessions” are compute runs/turns; they are not user-facing by default (surfaces “continue” by targeting a continuity and spawning runs behind the scenes).
- Background/subconscious agents are just jobs over event streams (summarizers, indexers, auditors, subagents). They must emit structured events + artifacts; no hidden mutable state.
- Multi-actor/shared continuities are first-class: every input/action must carry provenance (`actor_id`, `origin`) so team workflows remain replayable and auditable.

Agent mindset (avoid monotony)
- Optimize for the end-state architecture (complexity is allowed); implement in slices but **never** introduce “temporary” concepts that fight the Continuity OS model.
- For any feature, explicitly sanity-check: 1M+ events, provider cursor rotation, parallel jobs, multi-actor/shared continuities, remote control plane, replay determinism, and surface parity.
- When you notice a mismatch between docs/vision and implementation, treat it as a defect: fix docs and/or add a roadmap item in the same change.

Success metrics
- TTFT overhead, parse overhead per event, tool dispatch latency, patch throughput, end‑to‑end loop latency.
- CI gates must fail on regressions.

Scope boundaries
- Phase 1: core runtime, provider adapters, tool runtime, workspace engine, event log + snapshots, CLI (interactive + headless), agent server (HTTP/SSE + OpenAPI), benchmarks/fixtures.
- Phase 2: config/policy foundations, extensions/hooks, skills, subagents, context compiler + compaction, UI/interaction, SDKs, TUI/MCP surfaces, integrations, expanded execution modes + model/provider routing.
- Phase 3: search/index + memory, background workers/sync, policy/steering (adaptive budgets), enterprise config, extended sandboxing.

Architecture posture
- Rust core runtime for hot path.
- Internal compact frames; JSON only at edges.
- Plugins default to WASM; hot path may be native in‑process.
- Heavy modules may run out‑of‑process.
- All inter-module traffic is structured events, not raw text.

Surface parity principle
- No feature is considered done unless it exists in the core capability contract and is exposed in all active surfaces, or the gap is explicitly tracked and approved (owner + reason + expiry).
- Surface packages are adapters only; no business logic or core decisions live in UI or transport layers.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [numman-ali/rip](https://github.com/numman-ali/rip) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
