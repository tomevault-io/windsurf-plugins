---
trigger: always_on
description: Durable multi-agent orchestrator for web security testing. **Read this BEFORE modifying `event-bus.js`
---

# ARCHON — autonomous AI web-application pentester

Durable multi-agent orchestrator for web security testing. **Read this BEFORE modifying `event-bus.js`
or anything under `agents/` / `src/`.**

> **Layout:** personas live at `squads/<sq>/agents/<name>/` (universals at `_universal/agents/<name>/`),
> runtime state under `var/state/agents/<name>/`. **Never hardcode a persona path** — use `paths.js`
> (`agentPaths.soulPath(name)`, `skillsDir`, `personaState`, `lessonsPath`…). See the file table below.

## What this is

ARCHON is a Node.js daemon that runs LLM-powered specialist agents against a web target — **black-box**
(live URL), **static analysis** (source only), or **white-box** (both, merged into one de-duplicated
report). It runs on the **Claude subscription via OAuth — no API key** (`KURU_CLAUDE_BIN` → the `claude`
CLI). Two squads:

| Squad | Leader | Specialists | Domain |
|---|---|---|---|
| pentest | ATLAS | SCOUT, RANGER, TRACER (recon) · VIPER, DRILL, RELAY, VAULT, WARDEN, GATEWAY, SENTRY, KEYRING, LEDGER, FORGE, DECOY, SPECTRE (specialists) | Black-box web security |
| code-review | CURATOR | MARSHAL, CIPHER, QUILL, BEACON, BREAKER, SIPHON · PROBER (runtime validator) | White-box source review |

Universal agents (`_universal/agents/`): **AUDITOR** (independent verifier), **ARBITER** (confidence
judge), **SCRIBE** (final reporter), **COMMAND** (coordination), **TRIAGER** (dedup/merge + the
Findings board), **WRITER** (per-finding writeup). **NEXUS** is the daemon itself (`event-bus.js`),
not a persona.

## Top-level files

| File | Role |
|---|---|
| `event-bus.js` | The daemon (~11K lines). Dispatch queue → phased pipeline → report. **Be careful editing.** |
| `paths.js` | Persona/squad path resolver. Reads `layout.config.json` + `ownership.json` (mtime-cached, fail-soft). Portable roots via `KURU_AGENTS_ROOT` / `KURU_INTEL_ROOT` / `KURU_CLAUDE_BIN` + `.env.local` autoload. |
| `ownership.json` / `layout.config.json` | persona→squad-home map + layout knobs. |
| `scripts/dashboard.js` | The portal — HTTP API + SPA over the data layer (binds `127.0.0.1`). Dispatch/triage/report controls write to the daemon inbox. |
| `agents/runner/` | `agent-runner.js` = chokepoint `runAgent(spec)` → `{text,usage,model,raw}`; **default adapter `sdk`** (subscription OAuth), `ADAPTER=cli` rollback; `adapters/common.js` = env allowlist; `run-agent-bridge.js` = legacy `{code,output,cost,model}` shape for the spawnAgent retry path. |
| `src/dispatch/code-review-dispatcher.js` | White-box engine: inventories → App Blueprint → feature mapping → per-class assessment → AUDITOR → SCRIBE. |
| `src/core/squad-framework.js` | `SQUAD_TYPES` registry (pentest + code-review). |
| `src/routing/model-router.js` + `agents/model-config.js` | Per-agent model selection (`modelRouter.getModelForAgent(agentName, opts)`; family aliases via `modelRouter.resolveFamily(family)`). The family→model map is `<INTEL_ROOT>/model-config.json`; `agents/model-config.js` is the MODEL_PROFILE override shim. Never hardcode model strings. |
| `src/pipeline/` | Autonomy + methodology modules: `env-fingerprint`, `attack-planner`, `exploit-prover`, `outcome-classifier`, `cross-view-dedup`, `pentest-phases`, `chain-verifier`, `attack-graph`. |
| `src/core/coverage-map.js` + `common/taxonomy/owasp_wstg.yaml` | WSTG A-Z coverage map → specialist charters. |
| `src/pipeline/evidence-contract.js` | "No replayable evidence → not CONFIRMED" (enforced at `agents/auditor-validated-builder.js`). |
| `agents/scope-prevalidator.js` | Phase 0.0 scope gate — **fail-closed** (missing scope blocks; `ARCHON_SCOPE_OVERRIDE=1` to allow). |
| `agents/active-poc-policy.js` + `agents/active-poc-runner.js` | 3-gate perimeter for firing real payloads (engagement_mode + permission token + `ARCHON_ACTIVE_POC`). |
| `agents/finding-schema.js` | Canonical finding shape (`impact`, `proof_of_execution`, `reproduction_*`). |
| `common/` | `taxonomy/` (CWE + OWASP Top-10/API/LLM/Mobile + WSTG), `payloads/` (technique KB), `reporting/`, `remediation/`. Per-vendor WAF bypasses: `agents/refs/waf-bypass.md`. |

## Pipeline (pentest, `dispatchPentestParallel`)

All phases are **fail-soft** — an error logs and continues. Optional phases are config-gated via
`agents/squads/pentest/squad.json` `enabledPhases` (see `src/pipeline/pentest-phases.js`).

| Phase | Purpose |
|---|---|
| 0.0 | Scope pre-validate (fail-closed) |
| 0 / 0.7 | WAF detect + complexity scoring |
| 1 / 1.5 / 1.6 / 1.8 | Recon (SCOUT/RANGER/TRACER), spot-check, JS-bundle scan, EndpointModel |
| **0.6** | **Env fingerprint** — identify the exact product/stack + WAF vendor (`env-fingerprint.js`) |
| **1.9** | **Strategist** — ATLAS ranks a stack-specific attack plan, walking the WSTG map (`attack-planner.js`) |
| 2 | Specialist waves — generate **stack-specific** payloads, adapt to the WAF (fire→observe→mutate→refire) |
| 2.5 / 2.9 | Fast-verify + contradiction detector |
| 3 / 3.05 | AUDITOR verify → VALIDATED-FINDINGS (evidence contract enforced here) |
| 3.055–3.08 | Challenger, scope/prod annotation, evidence capture, severity filter, active-poc |
| **3.085** | **Exploit-Prover** — gated benign payload that *proves* impact → `proof_of_execution` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ghostshift-content/ARCHON](https://github.com/ghostshift-content/ARCHON) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
