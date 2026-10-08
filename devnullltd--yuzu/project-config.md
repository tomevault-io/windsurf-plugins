---
trigger: always_on
description: **This file is a contents page, not a knowledge base.** Before adding anything, read
---

# Yuzu — Codex / agent guide

**This file is a contents page, not a knowledge base.** Before adding anything, read
`docs/instruction-file-standard.md` — it defines where a rule belongs and why the default is *not
here*. This file, `CLAUDE.md`, and the four routed-concern tables load into every session; each is
budgeted at 40,000 characters and capped at 48,000 (`tests/test_issue_docs.py`).

This file is the Codex/Kimi twin of `CLAUDE.md` and deliberately carries the same content. **Where
they disagree, `CLAUDE.md` wins** — and the disagreement is a bug to fix in the same change.

## What is Yuzu?

An agentic enterprise endpoint management platform — a single control plane where agentic colleagues
query, command, scan, patch, and enforce policy compliance on Windows/Linux/macOS fleets in real
time; an open-source alternative to commercial endpoint platforms, built in C++23.

Goal: match mature-platform capability (`docs/capability-map.md`) with modern architecture —
gRPC/Protobuf transport, Prometheus-native metrics, **PostgreSQL server substrate + SQLite on the
agent** (ADR-0006), and a compiler-stable plugin ABI.

## Target architecture

Operators (agentic AI · browser · REST API · automation) → **Yuzu Server** (REST API v1, HTMX
dashboard — server-rendered and **dark-theme-only**, Instruction Engine, Policy Engine, Response /
Scope / RBAC / Audit, Scheduler, `/metrics`, Management Groups) → gRPC/Protobuf/mTLS bidirectional
streaming → **Yuzu Agent** per endpoint (Plugin Host over a **stable C ABI**, Trigger Engine, SQLite
KV, hash-verified content distribution, user interaction on Windows, per-plugin metrics).

Full component reference: `docs/architecture.md`.

## Glossary — three meanings of "agent"

Use the disambiguated form in commits, PRs, and new docs:

- **Agent daemon** — the C++ binary in `agents/core/` on each managed endpoint (the usual meaning).
- **Governance agent** — the `.claude/agents/*.md` review actors run during `/governance`.
- **Agentic worker** — an external LLM-driven client driving Yuzu via MCP, REST, or the dashboard
  (`docs/agentic-first-principle.md`).

## Context discipline

Context is the scarce resource. Keep this thread lean; spend file-reading budget in subagents.

- **Search before reading.** Grep (it *is* ripgrep) honours `.gitignore`; Read / Glob / `find` do
  **not** — keep those out of build, vendored and generated trees (`*.pb.h/.cc`, `vcpkg_installed/`).
- **Delegate broad reads.** Unknown locations or many-file sweeps → an `Explore` (or
  `general-purpose`) subagent that returns conclusions, not file dumps.
- **Summarise and hand off proactively** — record files touched, facts learned, open questions, next
  command; then continue or spawn a fresh agent.
- **For code changes:** owning module → its tests → smallest coherent patch → targeted suite
  (`meson test -C build-<os> --suite <agent|server|tar>`) before the broad one.

## Agent team & governance

Specialized agents live in `.claude/agents/` (each declares its role, triggers, reference docs).
`workflow-orchestrator` owns the gate sequence; **`/governance <range>` is the entry point** — never
hand-run the gates.

Pipeline (8 gates): Change Summary + Resource Ledger → security-guardian + docs-writer →
domain-triggered (`cpp-expert` + `cpp-safety` on any C++) → happy-path + unhappy-path +
consistency-auditor → chaos-injector → compliance-officer + sre + enterprise-readiness → findings
addressed → iterate.

**Four standing rules, all catastrophic-if-violated:**

1. **Routed-concern triggers are UNCONDITIONAL** — open the tables and match row by row against the
   changed paths; never work from recall. Diff size decides WHICH agents, never WHETHER.
2. **Severity is DERIVED, not chosen** — from a TRIGGER, an IMPACT, every applicable EXPOSURE, and an
   EPISTEMIC STATUS. BLOCKING = the derived band is CRITICAL or HIGH. **Policy floors** gate as
   contract violations and bypass the derivation entirely.
3. **`docs-writer` owns WORDING; the DOMAIN agent owns TRUTH.** A comment contradicting the code is a
   truth finding at native severity. A *missing* required doc is a third category — a truth finding,
   never capped at NICE.
4. **Gate 8 re-runs every gate whose DOMAIN THE FIX DIFF TOUCHES** — not only those whose findings
   prompted the fix.

**All four rules, the ledger schema, the run-ledger contract, and the enumerated list of files that
merely POINT at them are defined ONCE — in `.claude/skills/governance/SKILL.md`.** This file loses on
conflict, and so does `.codex/skills/governance/SKILL.md`, which defers there for severity and the
ledger. That runner does carry its own text for rules 1 and 4: it performs **no routed-concerns walk
at all** (tracked as **#2684** — which is why the instruction above to open all four tables by hand is
load-bearing on this leg), and its Gate 8 phrasing is the weaker "affected gates" wording that
shipped a broken macOS leg on #2580. The skill is read from your **working tree**, so a branch predating a change to it, or to
the routed-concern tables, silently runs the old pipeline; Step 0 opens with a per-file currency
check against `origin/dev`. The same rule governs any claim that a file or row is *absent* — verify

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DevNullLtd/Yuzu](https://github.com/DevNullLtd/Yuzu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
