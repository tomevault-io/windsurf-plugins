---
trigger: always_on
description: CLAUDE.md is the **team-collaboration kernel** — only rules that govern how the team (humans + AI) collaborate on this project: daily principles, the phase-entry workflow that loads the relevant rule subset on demand, and the Linux-first dev environment. Architecture-of-record, governance machinery, contract enforcement, and constraint mappings have moved to dedicated modules loaded by phase-contract skills; see *Where else to look* below.
---

# CLAUDE.md — Team Collaboration Kernel

CLAUDE.md is the **team-collaboration kernel** — only rules that govern how the team (humans + AI) collaborate on this project: daily principles, the phase-entry workflow that loads the relevant rule subset on demand, and the Linux-first dev environment. Architecture-of-record, governance machinery, contract enforcement, and constraint mappings have moved to dedicated modules loaded by phase-contract skills; see *Where else to look* below.

CLAUDE.md is **NOT** the architecture-of-record (read `architecture/workspace.dsl` + `architecture/docs/L0/ARCHITECTURE.md` on-demand via `/design-mode`). **NOT** the constraint corpus (`ARCHITECTURE.md` §4). **NOT** the runtime contract surface (`docs/contracts/contract-catalog.md`). **NOT** the L1 module design (`architecture/docs/L1/<module>/`). **NOT** the rule encyclopedia (`docs/governance/rules/*.md` are loaded on-demand by phase contracts; gate Rule 68/69 treat cards as the sole rule authority since 2026-05-28).

## Governance scope — main-path only

Governance (gates, ADRs, enforcers) constrains the engineering **main-path** only: product code, the architecture-of-record, runtime contracts, and the small set of current governed invariants. The gate's corpus scans exclude `docs/logs/` (history); governance applies only when justified.

## Phase Entry — Invoke the matching skill BEFORE working

ADR-0098 (rc21) replaces progressive on-demand rule loading with scenario-loaded contracts. At phase entry, **MUST invoke** the matching skill; the skill reads the phase contract and surfaces its active rules + forbidden patterns + exit criteria into context.

| When you are about to … | MUST invoke | Loads contract |
|---|---|---|
| Write ADR / module spec / SPI declaration / design review | `/design-mode` | [`architecture-design.md`](docs/governance/contracts/architecture-design.md) |
| Write production Java / yaml / Flyway migration / DI wiring | `/impl-mode` | [`engineering-implementation.md`](docs/governance/contracts/engineering-implementation.md) |
| Run gate / Maven verify / smoke / debug a regression | `/verify-mode` | [`integration-verification.md`](docs/governance/contracts/integration-verification.md) |
| Write release note / lockstep baseline / pre-commit checklist / open PR | `/commit-mode` | [`system-commit.md`](docs/governance/contracts/system-commit.md) |
| Process reviewer findings / corpus sweep / write rebuttal | `/review-mode` | [`review-response.md`](docs/governance/contracts/review-response.md) |

If uncertain which phase applies: default to `/impl-mode` (widest coverage). Skills suggest the next phase at exit.

## Daily collaboration principles

#### Rule D-1 — Root-Cause + Strongest-Interpretation Before Plan

**Before writing any plan, fix, or feature — surface assumptions, name confusion, and state tradeoffs. Then (a) name the root cause mechanically and (b) choose the strongest valid reading of the requirement.**

Enforced by [`rule-D-1.md`](docs/governance/rules/rule-D-1.md).

---
#### Rule D-2 — Simplicity & Surgical Changes

**Minimum code that solves the stated problem. Touch only what the task requires.**

Enforced by [`rule-D-2.md`](docs/governance/rules/rule-D-2.md).

---
#### Rule D-3 — Pre-Commit Checklist + Evidence-First Debug

**Before every commit, audit every touched file; fix defects before committing — "I'll fix it later" is forbidden; **smoke + lint** required before commits touching server entry points, runtime adapters, or dependency-wiring modules (sub-clause .a — Pre-Commit Checklist). When a Run fails, a test regresses, or a self-audit finding is opened, the first artefact captured MUST be observable evidence — failing test class FQN, trace ID, MDC slice (runId, tenantId, fromStatus→toStatus), and raw error message including stack frame line numbers; ARCHITECTURE.md / ADR consultation is permitted only AFTER evidence is recorded; self-audit findings under Rule D-5 that omit evidence citation are blocked (sub-clause .b — Evidence-First Debug; operationalised by `docs/harness/debug-first-evidence.md`).**

Enforced by [`rule-D-3.md`](docs/governance/rules/rule-D-3.md).

---
#### Rule D-4 — Three-Layer Testing, With Honest Assertions

A feature is implementable only when all three layers are designed. A feature is shippable only when all three are green and Rule D-5 passes.

Enforced by [`rule-D-4.md`](docs/governance/rules/rule-D-4.md).

---
#### Rule D-5 — Self-Audit is a Ship Gate, Not a Disclosure

A self-audit with open findings in a downstream-correctness category **blocks delivery**.

Enforced by [`rule-D-5.md`](docs/governance/rules/rule-D-5.md).

---
#### Rule D-9 — No Version / Log Metadata in Code


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chaosxingxc-orion/spring-ai-ascend](https://github.com/chaosxingxc-orion/spring-ai-ascend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
