---
trigger: always_on
description: ![Governance: Fail-Closed](https://img.shields.io/badge/Governance-Fail--Closed-critical)
---

# AGENTS.md — ADAAD Governed Build Agent

![Governance: Fail-Closed](https://img.shields.io/badge/Governance-Fail--Closed-critical)
![Agent: Governed](https://img.shields.io/badge/Agent-Governed-a855f7)
![DEVADAAD: Merge-Authorized](https://img.shields.io/badge/DEVADAAD-Merge--Authorized-ff6600)

> Governed automation contract for DUSTADAAD · Innovative AI LLC.

**Environment:** DUSTADAAD · Innovative AI LLC
**Trigger keywords:** `ADAAD` (build) · `DEVADAAD` (build + merge)
**Agent type:** Autonomous repository build agent — governed, fail-closed, evidence-producing
**Version:** 2.0.0
**Last reviewed:** 2026-03-07

> [!IMPORTANT]
> **Automation source-of-truth:** `docs/governance/ADAAD_PR_PROCESSION_2026-03-v2.md` is the controlling source for active sequencing and closure state. It supersedes `ADAAD_PR_PROCESSION_2026-03.md` (Phase 6 era, now archived).

---

## Trigger Contract

### `ADAAD` — Standard Build Trigger

`ADAAD` alone or as the first token activates the governed build workflow. Stages work for human review. **Does not merge.**

| Invocation | Effect |
|---|---|
| `ADAAD` | Continue from next unmerged PR in sequence |
| `ADAAD status` | Orientation report only; no build action |
| `ADAAD PR-XX` | Target specific PR; verify dependencies first |
| `ADAAD phase N` | Scope session to Phase N PRs only |
| `ADAAD preflight` | Preflight checks only; no build |
| `ADAAD verify` | Full verify stack against current state; no new code |
| `ADAAD audit` | Surface all open findings from `.adaad_agent_state.json` |
| `ADAAD retry` | Retry last blocked step after operator remediation |

---

### `DEVADAAD` — Authorized Merge Trigger

> **Operator-granted merge authority.** First word of prompt must be `DEVADAAD` exactly.
> All `ADAAD` build constraints apply in full. Merge is the only additional capability granted.

`DEVADAAD` activates the full build workflow **plus** conditional merge authority.
Merge is executed **only after every gate listed in the Merge Authorization Gate Stack passes with zero failures.**
A single gate failure at any tier blocks merge unconditionally — no exceptions, no overrides.

| Invocation | Effect |
|---|---|
| `DEVADAAD` | Build + merge next eligible PR if all gates pass |
| `DEVADAAD PR-XX` | Target specific PR; build + merge if all gates pass |
| `DEVADAAD dry-run` | Full gate stack evaluation; report results; do NOT merge |
| `DEVADAAD status` | Orientation report only; no build, no merge |
| `DEVADAAD audit` | Surface open findings; no build, no merge |

**`DEVADAAD` does not:**
- Merge code that has any failing test, replay divergence, lint error, or incomplete evidence row
- Override `INVARIANT PHASE6-HUMAN-0` — amendment proposals still require a recorded `human_signoff_token`
- Bypass branch protection rules on the repository platform
- Skip any tier of the gate stack
- Merge if the merge SHA differs from the verified SHA (re-gates required)

---

## Merge Authorization Gate Stack

> **Applies exclusively under `DEVADAAD`. All tiers are mandatory. Evaluated in order. Any failure → `[DEVADAAD MERGE-BLOCKED]` — stop.**

### Pre-Merge Gate Sequence

```
┌─────────────────────────────────────────────────────┐
│  TIER 0  — Baseline (schema, snapshot, lint)        │
│  TIER 1  — Full test suite + governance tests       │
│  TIER 2  — Replay + evidence suite (critical PRs)   │
│  TIER 3  — PR completeness (evidence, docs, lane)   │
│  TIER M  — Merge-specific gate (see below)          │
└─────────────────────────────────────────────────────┘
         All 5 tiers: PASS → MERGE AUTHORIZED
         Any single failure → MERGE BLOCKED
```

### Tier M — Merge-Specific Gate

Evaluated after Tiers 0–3 all pass.

| Check | Command / Condition | Blocks on |
|---|---|---|
| Working-code assertion | All `pytest` tests green on **merge SHA** (not branch tip) | Any failure |
| Zero test skips in scope | No `pytest.mark.skip`, `xfail`, or commented-out tests in changed files | Any found |
| Determinism re-verification | Re-run `lint_determinism.py` on merge SHA diff | Any violation |
| Replay digest match | Merge SHA replay digest == PR-staged digest | Any divergence |
| Evidence row `Complete` | `validate_release_evidence.py --require-complete` on merge SHA | Incomplete row |
| No pre-existing failures introduced | Diff of test results: merge SHA vs. base SHA | Any regression |
| Amendment human-signoff check | If PR touches `roadmap_amendment` paths → `human_signoff_token` must be present in ledger | Missing token |
| Merge attestation write | Agent writes `merge_attestation.v1` ledger event before pushing | Write failure |

### Merge Attestation Event (Required)

Before any merge is executed, the agent **must** write the following ledger event. If the write fails, merge is aborted.

```json
{
  "event_type": "merge_attestation.v1",
  "pr_id": "<PR-ID>",
  "merge_sha": "<merge_commit_sha>",
  "tier_0_digest": "<sha256 of Tier 0 output>",
  "tier_1_tests_passed": <N>,
  "tier_1_tests_failed": 0,
  "tier_2_replay_digest": "<digest | null>",
  "tier_3_evidence_complete": true,
  "tier_m_working_code": true,
  "triggered_by": "DEVADAAD",
  "operator_session": "<session_id>",
  "timestamp_utc": "<iso8601>",
  "human_signoff_token": "<token | null>"
}
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [InnovativeAI-adaad/adaad](https://github.com/InnovativeAI-adaad/adaad) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-26 -->
