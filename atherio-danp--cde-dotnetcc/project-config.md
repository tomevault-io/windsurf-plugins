---
trigger: always_on
description: > **TEMPLATE — fill in `{{ProductName}}` and this intro for your product.** This repo is a
---

# {{ProductName}}

> **TEMPLATE — fill in `{{ProductName}}` and this intro for your product.** This repo is a
> reusable Claude Code + .NET harness: Next.js (App Router) frontend + .NET 10 Web API backend,
> with an optional agentic pipeline on the Microsoft Agent Framework (MAF). Vision and full spec:
> `docs/product-overview.md` (also a template — fill it in).

> **Status: harness / governance only — no product code yet.** This repo holds the project
> standards and the Claude Code setup (rules, skills, agents, workflows, hooks). The `apps/web`
> (Next.js) and `apps/api` (.NET) trees are **not yet scaffolded** — governance files at
> `apps/api/` sit ready for when they are. After copying the harness into a new repo, find/replace
> `{{ProjectName}}` (namespaces / solution name) and `{{ProductName}}` (product identity) before
> building, and fill in the product docs.

## Ways of working

How we collaborate. These govern every action — follow them by default, every session.

**Decisions are Dan's.**
- **Never assume or decide on Dan's behalf.** When a decision is needed or anything is
  ambiguous, STOP and ask — present concrete options with a recommendation, don't pick silently.
- Don't proceed past a fork in scope or approach without explicit confirmation.
- When Dan pushes back, treat it as signal — re-examine, don't get defensive.
- **Lead with the product vision, not the tech.** When planning or grilling an idea, the
  "why / who / moat" comes first; the stack is the substrate, not the headline.

**Git & irreversible actions — ask first, every time.**
- **Never `git commit`, `push`, `amend`, switch branches, or open a PR unless Dan explicitly
  asks** — per action. Approval for one (e.g. "commit") never implies another (e.g. "push").
- Never force-push or rewrite shared history.
- Never delete or overwrite a file you didn't create without flagging it first.

**Stay in scope.**
- Do only what was asked. No "while I'm here" edits, no unrequested refactors.
- Don't over-engineer: no speculative abstractions or defensive code for cases that can't happen.
- Match existing patterns.
- **No third-party library without Dan's explicit, per-library approval.** Never introduce, assume,
  or default to a NuGet/npm package — prefer the BCL/framework or a minimal hand-rolled solution.
  Library adoption is Dan's call, never an agent's or a skill's; approval for one package never
  implies another. When something seems to need a library, STOP and propose it (with a no-library
  alternative) rather than adding it.

**Be honest and verify.**
- Evidence over assertion: show the command/test output; never claim success without verifying.
- No "probably" / "should work" — verify it, or label it explicitly as unverified.
- Use first-party/authoritative sources, quote them, and flag uncertainty.
- Report failures plainly: failed tests, skipped steps, dead ends.
- **Source-of-truth authority hierarchy — when sources conflict, the higher rung wins, ALWAYS:**
  1. **Code** — the running source is ground truth. Read it line-by-line; cite `file:line`. Never assert behaviour you haven't read.
  2. **Database / SQL data** — actual rows + schema over what code or docs *say* they are (query it; don't speculate on data state).
  3. **Telemetry** — observed runtime behaviour over described behaviour.
  4. **Documentation** (docs/, CLAUDE.md, rules, skills, memories) — a *lead to verify*, never an authority; may be stale. If docs and code disagree, the **code wins** — note the drift.
  5. **AI output** (other agents' or your own earlier output) — lowest; re-derive from a higher rung before trusting it.
- Make no claim you can't cite to a rung 1–3 source. If you can't verify, say "unverified" / "I don't know" — never fabricate.

**Safety.**
- Never read into context, or commit, secrets / `.env` / credentials.
- Never disable or skip tests, linters, or analyzers to make something pass — fix the root cause.
- No destructive shell (`rm -rf`, `git reset --hard`, …) without explicit confirmation.

**Workflow.**
- For non-trivial or multi-file changes, plan first and get the plan approved before coding. Write the plan
  to `docs/plans/<topic>.md` in the house format (`docs/projectStandards/implementation-plan-format.md`):
  locked decisions, an ordered checklist ending in a Validate gate, full code samples, an exact named-test
  list, OPEN QUESTIONS, and status banners with exact build/test counts.
- To build an approved plan, run `/run-impl-loop <plan>` — the main agent drives analyze → implement →
  validate → test → architect-review → triage → fix → summary, delegating the mechanical stages to the
  `impl-build` and `architect-review` workflows (see the `run-impl-loop` skill).
- Keep changes small and reviewable.

## Tooling — MCP servers & docs (non-negotiable)

Hard rules, not suggestions. They override default tool habits.

**Serena MCP — for all C# (`.cs`) work.** Use Serena for navigating, reading, searching,
editing, **and creating** `.cs` files — never the native Read/Grep/Edit/Write on them. A
`PreToolUse` hook (`.claude/hooks/enforce-serena.ps1`) **blocks native Edit/Write/MultiEdit
on `.cs`**, so C# edits/creates must go through Serena. The **TypeScript/React frontend

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [atherio-danp/cde-dotnetcc](https://github.com/atherio-danp/cde-dotnetcc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-18 -->
