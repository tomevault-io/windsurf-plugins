---
trigger: always_on
description: > This is the init file for this repository. It governs any AI agent (Claude Code, Codex, Cursor, etc.) working here.
---

# CLAUDE.md — Doberman Operating Manual

> This is the init file for this repository. It governs any AI agent (Claude Code, Codex, Cursor, etc.) working here.
> It is the **HOW**. The **WHAT** — the feature roadmap and per-slice detail — lives in `README.md` (public summary)
> and, if present in your working tree, the project development plan you keep alongside it.
> Content is identical to `AGENTS.md`; keep the two in sync (or symlink one to the other).
> If this file conflicts with a casual prompt instruction, **this file wins** unless a human overrides it in writing.

---

## 0. On startup (every session)

1. Read this entire file, then skim `README.md` for the current feature set, versioning, and roadmap. If you keep a
   local development plan alongside the repo, read the relevant slice there for the extra detail (objective, files,
   edge cases, suggested tests, suggested commit message).
2. `git status` + `git log --oneline -5`.
3. Pick the **next slice** (the lowest-numbered unmerged slice in the planned order).
4. If the repo lacks scaffolding (`pyproject.toml` / `.github/workflows/ci.yml`), do **Slice 0 (Bootstrap)** first (§6).
5. Execute exactly **one slice** via **The Slice Loop** (§5), then **STOP** and report. Stop at every review checkpoint.

---

## 1. What this repository is

**Doberman** is an adaptive authorization layer for coding agents, released publicly under **Apache-2.0**.

It distributes as the `doberman` package (`src/doberman/`) and is designed to be **genuinely functional on its own** —
schema, interfaces, the runtime harness, adapters, and the built-in rules all live here. It is **not** crippleware.

Doberman is built to be **extensible**: it declares stable interfaces (`Rule` / `Detector` / `AuthProvider` /
`AuditSink`) and a runtime registry that discovers implementations via **Python entry points**. Additional packages can
register their own rules, detectors, auth providers, or audit sinks **without Doberman importing them by name** — the
core never takes a static dependency on any plugin. With only `doberman` installed, it works (built-in protection);
install a plugin package and its capabilities light up automatically.

---

## 2. Architecture & extension points

The decision path is deliberately layered so the safety-critical core stays small, open, and auditable:

- **Tool mediation (`doberman.proxy`)** — the chokepoint. Every tool call an agent makes is normalized into a
  `SecurityObject` and routed through the decision engine. There is no path around it.
- **Decision engine (`doberman.engine`)** — combines guardrail verdicts into a final **allow / authenticate / block**
  decision. The execution rule and the raise-only `combine` are the safety invariants — they must stay open and
  auditable.
- **Objective guardrail + built-in rules (`doberman.engine.rules`)** — deterministic rules over the action: path
  canonicalization & confinement, destructive-command detection, external-destination checks, basic secret-pattern and
  encoded-exfil detection. New rules plug in through the `Rule` interface and the registry.
- **Roles & boundaries (`doberman.roles`)** — the role schema and per-repo boundary matcher.
- **Policy (`doberman.policy`)** — the default checklist and the strength modes (Light / Balanced / Strict / Paranoid).
- **Storage & audit (`doberman.storage`)** — a local, redacted decision log plus the `AuditSink` interface so
  additional sinks can be registered.
- **Tiered auth (`doberman.auth`)** — local confirmation, TOTP 2FA, and narrow/temporary role elevation. Auth providers
  plug in through the `AuthProvider` interface.
- **Subjective guardrail & baseline (`doberman.engine`)** — the abnormality interface plus a basic local behavioral
  baseline. Detectors plug in through the `Detector` interface.
- **Policy-drift & poisoning defense (`doberman.learning` / `doberman.policy`)** — classifies policy changes as
  strengthen vs weaken, gates weakening behind 2FA, and records changes in an append-only ledger. A core safety
  invariant: nothing auto-loosens.

**The plugin pattern (how to keep things decoupled):** declare the interface and registry in core; let other packages
**register** implementations through their own `pyproject.toml` entry points (groups such as `doberman.rules`,
`doberman.detectors`, `doberman.auth_providers`, `doberman.audit_sinks`). At runtime Doberman runs its built-in
implementations **plus** whatever is registered. Build the interface + a built-in implementation first; an advanced
implementation can then ship as a separate plugin package that depends on the now-merged extension point — never the
other way around.

---

## 3. Prime directives (non-negotiable)

These override everything. If a task would break one, **STOP and ask a human**.

1. **Fail closed.** On any error, uncertainty, or unhandled case → deny / `BLOCK`. The protected agent must never reach a tool around Doberman.
2. **Raise-only.** Guardrails/learning may auto-tighten; they may **never** silently loosen. Any weakening goes through the human-approved path (the policy-drift defense).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fu351/Doberman-Core](https://github.com/fu351/Doberman-Core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-19 -->
