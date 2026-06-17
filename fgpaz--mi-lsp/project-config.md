---
trigger: always_on
description: **Shared AE Gateway & Orchestration:** See `PATHS.md` for the authoritative AE Programa Gateway and Subagent Orchestration Protocol sections. This document covers agent/harness-specific workflow rules and enforcement semantics.
---

# mi-lsp Agent Policy

**Shared AE Gateway & Orchestration:** See `PATHS.md` for the authoritative AE Programa Gateway and Subagent Orchestration Protocol sections. This document covers agent/harness-specific workflow rules and enforcement semantics.

## Orchestration Mode (MANDATORY - Always Active)

For every task in this repository:

1. Run `$ps-contexto` first.
1a. Run `$ae-programa` as the gateway for non-trivial, mutating, policy, harness, shared-skill, or multi-step work.
1b. Select workers through manifest-backed global `ae-adapter-*` skills first: read `adapter_manifest.schema=ae-harness-adapter/v1`, prefer an explicit user-requested harness, then current/project harness fit, and fall back to `simulated_packets` with `missing_ae_adapter_manifest` when no adapter satisfies evidence and isolation.
1c. Worker-first is mandatory for AE-governed T2+, mutating, multi-step, policy/harness/shared-skill, runtime/deployable, or independent-axis work: use `worker_decision=spawned` when a usable adapter exists. `worker_decision=none` is valid only for `C0_INLINE_NO_DIFF` true read-only/no-diff work with no independent axes; `why_no_worker` is blocker evidence only.
1d. `ae-adapter-hermes` and `ae-adapter-claude-code` are discoverable partial seeds only; they are not usable until a native `ae-adapter-proof/v1` proves spawn, monitor, join, fallback, evidence, and sanitization.
1e. Non-trivial `.docs/auditoria/<session>/` folders must include `audit-manifest.yaml` with `schema: ae-audit-hygiene/v1`, `retention_ttl_days: 14`, `hash_algorithm: sha256`, artifact classes, sanitized summaries/verdicts, and cleanup status.
2. Validate governance before planning or execution:
   - set `MI_LSP_CLIENT_NAME` and `MI_LSP_SESSION_ID`
   - `mi-lsp workspace status <alias> --format toon`
   - `mi-lsp nav governance --workspace <alias> --format toon`
3. If governance is blocked, docs are not ready, `doc_count=0`, attribution is missing/manual, or `ae_canon` is not repo-canon valid, only diagnosis and repair are allowed until the repo is valid again.
4. After context load, run `$brainstorming` exactly once before planning or execution.
5. Close critical context gaps before acting.
6. Work in orchestrator mode by default.
7. Prefer `dispatching-parallel-agents` when work is safely partitionable.
8. Run `$ps-trazabilidad` before closing the task.

Additional strict rules:

- Spec-driven development is mandatory in ALL tasks.
- `.docs/wiki/00_gobierno_documental.md` is the human authority for governance.
- `.docs/wiki/_mi-lsp/read-model.toml` is the versioned executable projection of `00`.
- Do not push directly to `main`; create a branch and integrate through the PR flow. The PR flow is the integration mechanism, NOT a human-approval gate. Once the AE closure gates pass (`ps-trazabilidad` closure packet + `ps-auditar-trazabilidad` verdict `APPROVED` with all drift repaired + `scripts/ae/pre-push-guard.ps1` green + PR CI checks green), **auto-integrate the PR into `main`** via guarded merge without waiting for a separate human approval — `ps-auditar-trazabilidad` is the independent review. When branch protection requires a review and `enforce_admins=false`, complete it with an admin merge (`gh pr merge <n> --merge --admin --delete-branch`). Hold the PR open for a human only when the audit is `BLOCKED`, an `Approved with follow-ups` needs a human decision, a waiver is required, or the user explicitly asks to review. Never admin-merge over a failing CI check. Authority: `.docs/wiki/ae/AE-PHASES.md` (`AE-PHASES.integration_rule`).
- If governance is ambiguous, incomplete, out of sync, or the workspace index is stale relative to governance sources, the repo is in `blocked mode`.
- In `blocked mode`, only diagnosis and repair are allowed. Use `mi-lsp nav governance`, `$ps-asistente-wiki`, and `crear-gobierno-documental`.
- Run `$ps-auditar-trazabilidad` for large, risky, cross-layer, or multi-module changes.
- If editing `AGENTS.md` or `CLAUDE.md`, use `$ps-crear-agentsclaudemd`.
- Use `.docs/wiki/ae/` as the Agent Engineering layer. For workflow, policy, release, binary, worker, install, or publication work, enter through `$ae-programa`, load `AE-HARNESS-MANIFEST`, and close through `AE-RELEASE-DISTRIBUTION` when binaries can drift.
- If updating any skill under `C:\Users\fgpaz\.agents\skills`, also update the mirrored copy under `C:\repos\buho\assets\skills` in the same task.
- If creating or refactoring technical wiki docs under `07/08/09`, use `$crear-capa-tecnica-wiki`.
- If changing scope, architecture, or flows, use `crear-alcance`, `crear-arquitectura`, and `crear-flujo` in that order when applicable.

## Canonical Source of Truth (Project Paths)

Functional source of truth:

- `.docs/wiki/00_gobierno_documental.md`
- `.docs/wiki/01_alcance_funcional.md`
- `.docs/wiki/02_arquitectura.md`
- `.docs/wiki/03_FL.md`
- `.docs/wiki/03_FL/`
- `.docs/wiki/04_RF.md`
- `.docs/wiki/04_RF/`
- `.docs/wiki/05_modelo_datos.md`
- `.docs/wiki/06_matriz_pruebas_RF.md`
- `.docs/wiki/06_pruebas/`

Technical source of truth:

- `.docs/wiki/07_baseline_tecnica.md`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fgpaz/mi-lsp](https://github.com/fgpaz/mi-lsp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
