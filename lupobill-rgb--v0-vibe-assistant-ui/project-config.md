---
trigger: always_on
description: **Read this entire file before every session. Non-negotiable.**
---

# VIBE — Agent Operating Rules

**Read this entire file before every session. Non-negotiable.**

Last reviewed: 2026-04-15 — rewritten from the previous version after drift incidents traced to the previous file being deleted and rules going unenforced. If you are a Claude Code session, a planning session, or any other agent, and you have not read this file top to bottom in this session, stop and read it now.

---

## 1. Current Position

**Active sprint:** v7.1 build — three parallel tracks: (1) recommendation mode in edge function, (2) Build tab UI at `/build` route, (3) Salesforce + Slack connectors live + production validation of one end-to-end autonomous loop. Governing doc is `NORTHSTAR_v7_1.md`. v7.0 + Addenda superseded.
**Blocking:** Closing the autonomous loop end-to-end in production. Code path exists (autonomous_executions, processor, edge function autonomous detection) but no production-validated execution with customer-visible recommendation output. Edge Function currently routes autonomous mode to dashboard fast path — wrong shape, must route to new recommendation mode.
**Next to ship:** Recommendation mode in edge function (target Apr 22), then Build tab internal release behind feature flag (target Apr 25). See `NORTHSTAR_v7_1.md` Section 6 for full calendar.
**Last updated:** 2026-04-15

If any of these three fields is older than one week, stop the session and ask Bill to update them before proposing any work. A stale current-position field is the most common source of sessions that propose the wrong thing confidently.

---

## 2. Hard Stops — Things That Will Not Change Without Bill's Explicit Approval

Every rule in this section exists because something broke. The reason is load-bearing. If you read a rule and don't understand why it's a rule, ask before working around it.

### 2.1 Dashboard fast path is locked

**File:** `apps/api/src/index.ts`, lines approximately 808–839
**Behavior:** When `mode === 'dashboard'`, the job handler bypasses the planner and makes a single `edgeCall({ prompt, model, mode: 'dashboard' })` that returns a single-file HTML output.
**Why locked:** This exact pipeline produced the Advanced Decisions demo successfully. Every prior attempt to "improve" it has broken the demo pipeline and cost a half-day to a full day of repair.
**Allowed changes:** None without Bill's explicit approval in writing. New dashboard capabilities ship as new modes, not as modifications to this path.
**Enforcement status:** Discipline only. Needs CODEOWNERS gate. See Section 4.

### 2.2 VIBE_SYSTEM_RULES is the only system prompt for the dashboard fast path

**File:** `supabase/functions/generate-diff/index.ts`, near line 581
**Behavior:** The edge function prepends `VIBE_SYSTEM_RULES` to every mode's system message. Dashboard mode adds `DASHBOARD_SYSTEM` and design-phase specs. No other system prompts, no prompt wrappers, no middleware prompt injection.
**Why locked:** Prompt stacking changes dashboard output in ways that pass code review but fail visual inspection. Every layer added has produced a dashboard regression.
**Allowed changes:** None without Bill's explicit approval. If you believe the system prompt needs to change, the first step is running the smoke test (Section 4) against the proposed change and attaching the output to the PR.
**Enforcement status:** Discipline only. Needs CI smoke test gate on any PR touching this file.

### 2.3 Single-file HTML output for dashboard mode

**Contract:** The edge function returns `{ diff: "<html>..." }` — one self-contained HTML file. `index.html` is the only file written. `manifest.json` contains `["index"]`. No `<a>` navigation between pages, no multi-page routing.
**Why locked:** Multi-page dashboards were attempted and broke preview rendering. The single-file output is what the preview iframe expects.
**Allowed changes:** None without a new mode. Multi-page dashboards, if built, are a separate mode.
**Enforcement status:** Discipline only. Needs output shape check in CI.

### 2.4 User prompt passes directly, no rewriting

**Contract:** The `enrichedPrompt` (user prompt + org context) is sent as the user message. No rewriting, no summarization, no template wrapping of the user's intent.
**Why locked:** Prompt rewriting was attempted to "improve" dashboard quality and broke the Advanced Decisions demo. Reverted in PR #514.
**Allowed changes:** None without explicit approval. Context injection via the kernel is fine; rewriting the user's text is not.
**Enforcement status:** Discipline only.

### 2.5 v7.1 is the active spec; v7.0 and prior addenda are superseded

**Rule:** Work is governed by `NORTHSTAR_v7_1.md`, not v7.0 or its addenda. v7.0, Trust Layer Addendum, Addendum B, and v6.0 Addendum are archived in `/docs/archive/` for audit trail only.
**What's in v7.1:** Three parallel tracks — recommendation mode (edge function output shape), Build tab UI (single user surface for both autonomous outputs and manual builds), connector expansion + production validation (Salesforce + Slack live, one autonomous loop demonstrated end-to-end). Target: May 9.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lupobill-rgb/v0-vibe-assistant-ui](https://github.com/lupobill-rgb/v0-vibe-assistant-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-12 -->
