---
trigger: always_on
description: > **This file is read at the start of every Claude Code session. It is the
---

# CLAUDE.md — Plumbline Project Constitution

> **This file is read at the start of every Claude Code session. It is the
> single source of behavioral truth for building Plumbline. When any other
> instruction conflicts with this file, this file wins. When this file is
> silent, consult `/docs/adr` (decisions) and `/docs/specs` (component specs).**

---

## 0. What Plumbline is (in one paragraph)

Plumbline is an open-source **static analyzer for LLM and agentic AI applications**.
It is **not a linter** in the style-checking sense — it is a reliability and
architecture analyzer. It answers the question *"will this agentic system fall
over in production?"* by statically detecting reliability, architecture,
harness-engineering, and security defects in AI/agent code. A plumb line is the
craftsman's oldest instrument for checking whether a structure is built true;
this tool does the same for agentic systems. It is built by **ActaClad** and is
the design-time companion to ActaClad's runtime trust platform, **AgentGuard**.

---

## 1. Non-negotiable principles

These are invariants. Do not violate them. If a task seems to require violating
one, stop and surface the conflict rather than proceeding.

1. **Detection is deterministic. Remediation may use AI.**
   The rule engine is pure static analysis (AST + dataflow). Running Plumbline
   twice on the same code MUST produce identical findings. An LLM may ONLY be
   used to generate human-readable fix/remediation text — NEVER to decide
   whether a finding exists. No network calls in the detection path.
   Rule knowledge may also be exported as a generation-time skill-pack to
   *prevent* defects at authoring time (ADR-0011), but that pack assists
   authoring only — it is never the gate and never a substitute for the
   deterministic engine, which remains the sole verification authority.

2. **Dataflow over pattern-matching.**
   The core engine is a taint/dataflow tracker (untrusted sources → dangerous
   sinks). Grep/regex-style rules are the exception, used only where dataflow
   genuinely does not apply, and must be marked as such.

3. **Every finding carries a confidence level: High / Medium / Low.**
   - High = deterministic, ~90%+ precision, safe to fail a build.
   - Medium = strong heuristic, advisory by default.
   - Low = informational, EXCLUDED from scoring.
   A rule may not ship as High without a measured precision number in
   `/benchmark`. If precision is unknown or <90%, it ships as Medium or Low.

4. **False positives are the enemy.** A noisy analyzer gets uninstalled. When in
   doubt between catching more and being wrong less, choose being wrong less.

5. **One rule = one detector module + one passing fixture + one failing fixture.**
   No rule is "done" until it has both a vulnerable fixture (must trigger) and a
   clean fixture (must NOT trigger), both in `/fixtures`, both under test.

6. **The Quality Gate is the product; the score is the dashboard.**
   Teams wire the pass/fail Quality Gate into CI. Scores (pillar scores +
   composite) are stakeholder roll-ups. Never make the score the CI mechanism.

7. **Reliability leads. Security is a competent pillar, not the headline.**
   When prioritizing what to build or which rules to deepen, the order is:
   Reliability → Architecture & Agentic Maturity → Harness Engineering →
   Security. We deliberately do NOT compete as "another security scanner."

8. **Standards-grade output.** SARIF output is mandatory and must validate
   against the SARIF 2.1.0 schema. Every rule maps to external standards where
   they exist (OWASP LLM Top 10, OWASP Agentic Top 10, NIST AI RMF, CWE).

9. **Capture every architectural decision as an ADR.** Any non-trivial
   design choice gets a numbered, immutable ADR in `/docs/adr` BEFORE or
   ALONGSIDE the code that implements it. ADRs are never edited after
   acceptance — only superseded by a new ADR.

10. **Naming hygiene.** The composite score is the **Readiness Score** (or
    "Reliability Score") — NEVER "Trust Score." "Trust Score" and "Trust
    Profile" belong exclusively to AgentGuard. Plumbline FEEDS those; it does
    not reproduce them.

---

## 2. Workflow Claude Code must follow

Work in tight, reviewable increments. The loop for every unit of work:

```
spec  →  ADR (if a decision is made)  →  test (failing)  →  implement  →
green tests  →  update docs  →  commit (with ADR reference)
```

- **Read before writing.** At session start, read this file, then the relevant
  spec in `/docs/specs`, then any ADRs touching the area. Never implement a
  component whose spec you have not read.
- **Small diffs.** Prefer many small, coherent commits over large ones. A diff a
  human cannot review in 10 minutes is too big. The failure mode of agentic
  coding is large unreviewed diffs that work but nobody understands — actively
  prevent it.
- **Test-first for rules and core.** Write the failing fixture/test, then the
  detector. A rule with no failing fixture is not allowed to exist.
- **No scope creep.** If you discover work beyond the current task, write it to
  `/docs/backlog.md` and keep going. Do not silently expand scope.
- **Surface conflicts, don't paper over them.** If a spec, an ADR, and the code

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ActaClad/plumbline](https://github.com/ActaClad/plumbline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
