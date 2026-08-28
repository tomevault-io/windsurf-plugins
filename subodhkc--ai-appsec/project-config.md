---
trigger: always_on
description: > **Scope:** These rules apply ONLY to the `haiec-ai-agent-security-free-mcp` build session
---

# AGENTS.md — HAIEC Agent Security MCP (Phase -1 → Phase 18)

> **Scope:** These rules apply ONLY to the `haiec-ai-agent-security-free-mcp` build session
> (the multi-phase MCP construction project, ~16-18 phases). They do NOT apply to other
> projects on this machine. They are loaded automatically when this repo (or the
> `HAIEC-workspace/` parent) is the active workspace.
>
> **Persistence:** This file is the source of truth for session rules across phases.
> Do not delete or weaken it mid-project. If a later phase needs to amend a rule,
> edit this file explicitly and note the change in `PHASES.md`.

---

## 1. Workspace Layout

```
HAIEC-workspace/
├── haiec-ai-agent-security-free-mcp/   ← WRITE HERE (primary working repo)
├── haiec-website/                      ← READ ONLY (junction to ../Haiec Website)
├── llmverify-npm/                      ← READ ONLY (junction to ../llmverify-npm)
└── mcp-tenant-isolation/               ← READ ONLY (junction to ../mcp-tenant-isolation)
```

- **Primary write repo:** `haiec-ai-agent-security-free-mcp` (public, GitHub:
  `subodhkc/haiec-ai-agent-security-free-mcp`, default branch `main`).
- **Read-only repos** are Windows directory junctions to existing local clones.
  They share the working tree with those clones — inspect, do not modify.
- **No separate `llmverify/` repo exists.** `llmverify-npm` is the only LLMVerify
  repo in scope (user-confirmed 2026-08-16). Do not invent a `llmverify/` path.

---

## 2. Architectural Principles (apply to every phase)

1. **Not a wrapper around three scanners.** This repo becomes the public Agent
   Security control surface. LLMVerify and Tenant Isolation remain independent
   products and independent engines. Compose them ONLY where composition has an
   actual semantic reason. Never let the architecture drift into
   "run everything every time."

2. **Strict engine independence.** `scan_ai_security`, `scan_tenant_isolation`,
   `verify_llm_content`, and `check_deploy_security` NEVER automatically invoke
   one another. Each is selected on its own semantic merit.

3. **Source of truth is fragmented.** Conflicting scanner/rule/Semgrep version
   declarations have already been found inside HAIEC. Treat production executable
   code as evidence; treat older documents as hypotheses until verified. This is
   why Phase -1 exists.

4. **Evidence hierarchy:** executable source / tests / config > README / docs /
   comments. Never convert an assumption into a fact without evidence.

5. **Proactive AI use is a product acceptance criterion.** A feature is not
   finished because `npx` works. We must verify that realistic prompts cause
   Cursor/Claude/Windsurf/VS Code agents to choose the correct capability, AND
   that unrelated prompts correctly cause NO HAIEC invocation. **False
   invocation is just as important a defect as missed invocation.**

6. **Tool descriptions = semantic precision, not promotion.** Describe when to
   use AND when not to use each function. Avoid "best security scanner" /
   "most comprehensive" phrasing. The model must understand the ontology:
   - source-code security        → `scan_ai_security`
   - cross-tenant boundaries     → `scan_tenant_isolation`
   - actual LLM input/output     → `verify_llm_content`
   - merge/release/deploy        → `check_deploy_security`

7. **Scan Receipt and proof-of-fix are architectural primitives**, not optional
   marketing artifacts. Design them in from the start: reproducibility, AI
   repair loops, CI evidence, future cloud ingestion format, shareable outputs.

8. **Keep the architecture simple.** Do not rebuild HAIEC SaaS inside the
   open-source repo.

---

## 3. Hard Constraints (never violate)

- **Never modify, commit, push, publish, tag, release, migrate, or deploy
  anything unless the current phase prompt explicitly requests it.**
- **Never make opportunistic fixes while auditing.** Document them in
  `FINDINGS.md` (or per-phase findings file) and propose later fixes.
- **Read-only repos stay read-only** (`haiec-website`, `llmverify-npm`,
  `mcp-tenant-isolation`) unless a later prompt explicitly authorizes changes.
- **Local means local.** No silent cloud/network fallback.
- **Never execute target repository code during scanning.**
- **Never expose secrets / raw hostile repository content unnecessarily to AI
  context.**
- **Never bypass errors.** Fix root causes, not symptoms.
- **Preserve invariants on every fix.**
- **Critical flows must be sequential, not parallel.**
- **Frontend must not assume backend state.**
- **Backend must never fail silently at startup.**
- **Local state ≠ source of truth.**

---

## 4. Phase Discipline

- **At the start of each phase:** run the phase-entry checks BEFORE changing code.
- **At the end of each phase:** run the phase-exit gate BEFORE declaring
  completion. Do not declare done until the gate passes.
- **If evidence contradicts the current plan:** report it; do not force
  implementation to match the plan.
- **Track every phase** in `PHASES.md` (status, key decisions, evidence refs,
  findings). This is how context survives across 16-18 phases.
- **Context preservation:** before ending a phase, update `PHASES.md` with

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [subodhkc/ai-appsec](https://github.com/subodhkc/ai-appsec) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-27 -->
