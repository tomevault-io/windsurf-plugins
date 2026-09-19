---
trigger: always_on
description: Build the hackathon MVP described in `docs/ProofGate_PRD_FINAL_v4.md`.
---

# ProofGate Repository Instructions

## Mission

Build the hackathon MVP described in `docs/ProofGate_PRD_FINAL_v4.md`.

ProofGate measures the blast radius of consequential AI-agent actions before execution.

Do not broaden the product beyond the PRD.

---

## Python Environment

Before running Python-related commands, use the repository virtual environment.

Prefer:

```bash
source .venv/bin/activate
```

If activation persistence is uncertain across separate shell invocations, use the
explicit executables instead:

- .venv/bin/python
- .venv/bin/pytest
- .venv/bin/pip

Never install packages globally.

---

## Non-Negotiable Demo

The user requests:

> Clean up inactive test accounts that have not logged in for 90 days.

The simulated bad proposal is:

```python
delete_users(inactive_days=90)
```

It omits:

```python
environment="test"
```

The deterministic seed must produce:

- 9,981 inactive production users
- 92 inactive test users
- 10,073 total affected by the broad call

The protected flow must show:

1. Broad delete is `BLOCKED`.
2. No mutation occurs after `BLOCK`.
3. Only actually triggered policy rules are displayed.
4. Agent adds `environment="test"`.
5. Agent creates a selector-bound snapshot proof.
6. The same public `delete_users` tool is retried.
7. Corrected action is `ALLOWED`.
8. Predicted count = 92.
9. Actual count = 92.
10. Production affected = 0.
11. Postcondition status = `VERIFIED`.
12. Workflow budget = 92/100.

---

## Frozen Architectural Boundary

The public ProofGate boundary accepts policy context and proof:

```python
guarded_delete_users(
    action_context,
    inactive_days,
    environment,
    rollback_proof,
)
```

The internal Operations mutation stays dumb:

```python
operations.delete_users(
    inactive_days,
    environment,
)
```

`rollback_proof` belongs to ProofGate, never to the underlying mutation function.

Protected mode may call `operations.delete_users` only after ProofGate returns `ALLOW`.

Unprotected mode intentionally calls it directly.

Do not change this contract without explicit user approval.

---

## Architectural Invariants

- CRAFT is read-only enterprise intelligence.
- CRAFT does not perform deletion.
- SQLite shadow CRM is authoritative for exact mutation impact.
- Nebius extracts intent and structured risk features.
- Nebius must never return `ALLOW` or `BLOCK`.
- Deterministic Python policy makes every verdict.
- Rollback proof is recoverability, not authorization.
- Valid proof must never override an intent mismatch.
- Unknown impact for a consequential action fails closed.
- Workflow mutation budget is 100 rows.
- Production deletion budget is zero.
- Risk factors and triggered policy rules are separate concepts.
- The UI renders only rules that actually fired.

---

## Selector Hash Contract

All selector hashes must use one shared canonicalization function.

Canonical JSON:

```python
json.dumps(
    arguments,
    sort_keys=True,
    separators=(",", ":"),
    ensure_ascii=True,
)
```

Hash the resulting UTF-8 bytes using SHA-256.

The selector hash must include only mutation-selecting arguments.

Include:

- `inactive_days`
- `environment`

Exclude:

- `rollback_proof`
- `action_context`
- workflow metadata
- policy metadata
- risk metadata
- timestamps
- audit fields

For the corrected demo action, the selector is logically equivalent to:

```json
{
  "environment": "test",
  "inactive_days": 90
}
```

The same canonicalization function must be used by:

- impact preview
- snapshot creation
- proof validation
- guarded execution

Do not duplicate selector-hash logic across modules.

---

## Nebius Output Rules

Nebius is used for:

1. Intent extraction.
2. Structured risk-feature extraction.

Nebius responses must:

- use strict JSON
- match Pydantic models
- never invent counts
- use `ImpactEnvelope` counts as authoritative
- return semantic features only
- never return `ALLOW` or `BLOCK`
- have a deterministic regex fallback

Python policy code makes the final verdict.

---

## CRAFT Claim Discipline

Display CRAFT evidence separately from operational impact.

Use these labels:

- `CRAFT enterprise evidence`
- `Operations impact preflight`

CRAFT provides enterprise context and read-only analytical evidence.

The Operations preflight calculates the authoritative mutation blast radius.

Never imply cached evidence is live.

If cached evidence is used, label it:

- `Previously retrieved CRAFT evidence`

---

## Demo Interaction Discipline

The complete demo should require no more than:

1. Run Unprotected.
2. Reset and Run Protected.
3. Optional details expansion.

Snapshot creation is real but automatic.

The main UI may jump directly to a populated `PROOF VALID` panel.

Do not require separate user clicks for:

- repair
- snapshot creation
- proof validation
- corrected retry
- postcondition verification

---

## Safety

- All mutations target only `operations/working.db`.
- Never mutate `operations/pristine.db`.
- Never run destructive commands outside the repository.
- Do not print API keys, OAuth tokens, or complete environment variables.
- Do not commit `.env`, token caches, credentials, or generated secrets.
- Ask before deleting files.
- Ask before changing the frozen architecture.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Karthik6412/proofgate](https://github.com/Karthik6412/proofgate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-19 -->
