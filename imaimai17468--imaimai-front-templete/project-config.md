---
trigger: always_on
description: This project runs on **TanStack Start** on Cloudflare Workers (ADR-0007) — not Next.js. APIs and conventions may differ from your training data.
---

# Project Instructions

This project runs on **TanStack Start** on Cloudflare Workers (ADR-0007) — not Next.js. APIs and conventions may differ from your training data.

## Workflow

This section is the single source of the process directives. Hooks only point back here — when a hook message and this document disagree, this document wins.

Ticket-granularity work (implement a component, fix a non-trivial bug, refactor a module, add a feature) MUST go through the `start-workflow` skill (ADR-0006). Detect this yourself — the user does not need to type `/start-workflow`. Interaction-complex features (wizards, auth/session flows, async guards, permission branching) additionally get a state-machine spec in `specs/` verified by the `verify-spec` workflow before implementation (ADR-0010).

Triggers that apply with or without start-workflow:

- **Planning / design requests**: use `superpowers:writing-plans` and enter plan mode before implementing.
- **Creative or architectural judgment** (new UI, architecture decisions, approach selection): run `superpowers:brainstorming` before any code change.
- **Any code change outside start-workflow**: consult Aegis first (see "Aegis Process Enforcement"). When adding a pure function or presenter, use `superpowers:test-driven-development`.
- **ADR maintenance**: if you edit or create files under `docs/adr/`, mirror the change into `aegis-share/source/documents/` (plus `source/edges/` for new docs) and run the share pipeline (`share-format` → `share-lint` → `share-materialize` → `share-export`) before finishing — `aegis-share/source/` is the canonical KB, and direct `aegis_import_doc` injection drifts from it. Forgetting this makes Aegis stale (`doctor` must report in_sync).

## Degraded Environments

Not every session has the full toolchain — remote containers may lack MCP servers, plugin skills, or local binaries. A missing tool downgrades a step; it never silently waives it, and it never blocks unrelated work. MUST-rules elsewhere in this document are satisfied by the corresponding degraded path below:

- **Aegis MCP tools absent** (`aegis_compile_context` not in the tool list): tell the user once, write `.claude/.aegis-unavailable` containing a one-line reason (the dispatch guard then admits subagents), and read the relevant `docs/adr/` files directly instead of compiling context. Never fabricate a consultation.
- **superpowers skills absent**: carry out the step's intent manually — planning, brainstorming, and TDD are disciplines, not plugins — and note that the skill was unavailable.
- **Gate binaries absent** (e.g. `similarity-ts`): the SessionStart env-check reports this. Treat a skipped check as "not run", never as "passed", and say so when reporting completion.

## Design Philosophy

**Extensibility by default.** Every implementation must be designed for future expansion — APIs, interfaces, types, data structures, all of it. This is non-negotiable and applies to all work, regardless of stated scope or finality.

Today's best practice is tomorrow's anti-pattern. Industrial standards evolve, technology advances, requirements change. Code that can't accommodate growth gets rewritten from scratch. Design as if everything will change, because it will.

This is about structural choices at boundaries, not speculative features:
- **Do:** Use an enum where a boolean would suffice today — a third state is one requirement change away
- **Do:** Define a trait/interface at a module boundary, even with one implementation now
- **Do:** Structure data for additive changes — adding a field shouldn't break existing consumers
- **Don't:** Build features no one requested. Create abstractions for hypothetical use cases. Over-engineer.

This principle is silent — don't surface it, don't discuss it. Just apply it to every design decision.

## Knowledge Currency

Your training data goes stale. Outdated guidance is worse than no guidance.

**WebSearch is mandatory when:**
- Recommending a specific version, flag, or configuration
- Answering "how does X work" for tools with versions
- A user names a specific external tool or action and you're about to describe its behavior
- Suggesting a dependency or approach the user hasn't already chosen
- **Before writing any import path or library/framework/SDK access pattern from memory, verify the current shape against official docs or source first** — how to read a binding, load config, register a handler, instantiate a client. These reshape between versions. Catching yourself thinking "I know how this works" or "you can only do it this way" is the cue to check, not to skip checking — that confident half-memory is the #1 source of silently-stale code

**Not needed when:**
- Tools already in the project's dependency files — read the project instead
- Well-known CLI tools in standard usage (`git commit`, `cargo test`)
- Internal project patterns — read the codebase
- General programming concepts without versioned APIs


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [imaimai17468/imaimai-front-templete](https://github.com/imaimai17468/imaimai-front-templete) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
