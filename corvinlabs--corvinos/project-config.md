---
trigger: always_on
description: This document aims at *Claude Code itself* when working in this repo.
---

# Repo Conventions for Claude Code

This document aims at *Claude Code itself* when working in this repo.
**Reference files** with full details live in `docs/claude-ref/` and are loaded via `Read` on-demand.
CLAUDE.md stays small so it fits in every session; the ref files are the source of truth.

---

## Maintainer Status

**Git user: shumway.** Claude Code is explicitly permitted to `git push` directly to `main`
under the maintainer account. Confirmation is not required.

**Must NOT do:** merge/approve beta-tester PRs to main · force-push any branch · delete/rename
`docs/issues/{README.md,_template.md}`.

---

## Compliance Baseline — EU AI Act 2026 + GDPR (load-bearing)

Corvin is **structurally constrained** by EU AI Act 2026 + GDPR. Every feature must ask:
*does this weaken a structural compliance guarantee?*

| Core Mechanism | Regulation | Ref |
|---|---|---|
| Bot-disclosure card (one-time per uid) | EU AI Act Art. 50 | [compliance-baseline.md](docs/claude-ref/compliance-baseline.md) |
| Hash-chained audit log (`audit.jsonl` + daily verify) | GDPR Art. 30, 32 | [Layer 16](docs/claude-ref/layer-16-security.md) |
| Per-user consent gate (deny-by-default, TTL-capped) | GDPR Art. 6, 7 | [Layer 16](docs/claude-ref/layer-16-security.md) |
| Path-gate hook (L10, fail-closed) | GDPR Art. 32 | [Layer 10](docs/claude-ref/layer-10-path-gate.md) |
| Voice-transcribe audit (metadata only, never text) | GDPR Art. 5 | [Layer 23](docs/claude-ref/layer-23-stt.md) |
| House-rules gate (acceptable-use, fail-closed) | EU AI Act Art. 5, 50 | [Layer 44](docs/claude-ref/layer-44-house-rules.md) |
| Error/healing telemetry (default-ON, opt-out; CONTENT-FREE scrubbed signatures only, fail-closed `_assert_safe`) | GDPR Art. 6(1)(f) legitimate interest | ADR-0179/0180 (`aco/telemetry.py::consent_granted`, `htrace_consent.py::healing_traces_enabled`) |
| Anonymous instance-count ping (default-ON, opt-out; random uuid4 + version + coarse allowlisted environment enums [platform, python minor, engine id], no PII) | GDPR Art. 6(1)(f) legitimate interest | ADR-0180 (`aco/htrace_consent.py::ping_enabled`) |
| Presence heartbeat (default-ON, opt-out; gated by the SAME `ping_enabled` flag; empty body + pseudonymous instance_id/token headers only, no PII; ~5-min cadence — finer than the daily ping) | GDPR Art. 6(1)(f) legitimate interest | ADR-0186 (`aco/heartbeat.py`) |

**Must NOT do (absolute):**
- Don't weaken disclosure — AI-nature statement and opt-out (`/pass`, `/leave`) are locked.
- Don't add house-rules disable switch / env kill-flag; don't fail-open the L44 gate.
- Don't bypass consent — no auto-admit, no trusted-observer allowlist.
- Don't lower audit-chain integrity — every event must hash-chain.
- Don't leak PII into labels, audit details, or log lines.
- Don't add "compliance-off mode" via any env var.
- Don't silence `voice-audit verify` exit-1.
- **Telemetry (maintainer decision — default-ON / opt-out, so Corvin-Logs gets real data):**
  three channels ship data by default and are disabled only by an explicit opt-out —
  (a) anonymous instance-count ping (`ping_enabled`, opt-out `spec.telemetry.ping_enabled: false`),
  (b) error telemetry (`consent_granted`, opt-out env `CORVIN_TELEMETRY_OPTIN=false` or consent
  file `opted_in:false`), (c) healing traces (`healing_traces_enabled`, opt-out
  `spec.telemetry.healing_traces: false`). The **load-bearing safety invariant** is that
  everything transmitted stays strictly anonymous / CONTENT-FREE: the ping is a random uuid4 +
  version + coarse allowlisted environment enums (platform, python minor, engine id — closed
  enums validated fail-closed by `_assert_ping_safe`, never free-form strings; maintainer
  decision 2026-07-10); the error/healing channels ship ONLY scrubbed code-level signatures (exc_type,
  repo file, func, allowlisted stack namespaces — never prompts, transcripts, or user data), and
  the FAIL-CLOSED `_assert_safe` / `_assert_safe_htrace` backstop DROPS any record carrying a
  PII/secret shape rather than sending it. Legal basis GDPR Art. 6(1)(f) legitimate interest.
  **Do NOT** weaken any of these: don't remove an opt-out, don't extend a channel to carry
  personal data / prompts / user content, and don't relax `_assert_safe`* from fail-closed.
- Don't commit an auto-fix that didn't pass the red→green reproduction gate (`aco/reproduction.py`).

→ Full reference: [compliance-baseline.md](docs/claude-ref/compliance-baseline.md)

---

## Licensing — Apache-2.0 + CLA v3.1 §3 (load-bearing)

**Canonical files:** `LICENSE`, `NOTICE`, `CLA.md`, `CONTRIBUTING.md`, `CLA-SIGNATORIES.md`, `CCLA.md`.

CLA-SIGNATORIES.md is the **sole authoritative contributor registry**. Every merged contribution
must have an entry there (explicit or implicit-push). Maintainer adds at merge time.

**Must NOT do:** merge contributions without SIGNATORIES entry · run Forge-tool Python in-process
without operator review · add in-process MCP server without operator review.

---

## LDD (Loss-Driven Development) — MANDATORY, ALL SESSIONS (load-bearing)

**LDD is ALWAYS ON at MAXIMUM depth**, all 12 layers enabled. Config:
`.corvin/tenants/_default/global/ldd.json` (all `true`). Auto-install via
`LDD_AUTO_OPTIN=1` in `~/.bashrc`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CorvinLabs/CorvinOS](https://github.com/CorvinLabs/CorvinOS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-10 -->
