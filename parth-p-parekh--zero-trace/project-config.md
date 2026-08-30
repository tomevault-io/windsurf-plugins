---
trigger: always_on
description: This file applies to this directory and all child directories.
---

# ZeroTrace Agent Instructions

## 1. Scope

This file applies to this directory and all child directories.

Read this file before you inspect or change files.

The repository contains the ZeroTrace product.

`Control-DB/` contains the current Part A implementation.

`docs/` contains the product, engineering, scope, and scoring rules.

Do not assume that planned files exist.

Verify the current tree before you refer to a component as implemented.

## 2. Required response style

Agents MUST respond only in ASD-STE100 style.

Use short sentences.

Use common words.

Use active voice.

Use one instruction or fact per sentence.

Use one idea per paragraph.

Use clear headings and bullet lists.

Use exact file paths, symbols, commands, and test names.

Avoid idioms, slogans, marketing language, vague qualifiers, and unnecessary adjectives.

Do not use words that hide uncertainty, such as "probably", "basically", or "obviously".

State unknown facts as unknown facts.

Separate observed facts from inferences.

Use `Observed:`, `Decision:`, `Risk:`, `Check:`, and `Next:` when they improve clarity.

Explain important behavior from first principles.

For each first-principles explanation, state:

1. The input.
2. The rule or transformation.
3. The output.
4. The reason for the rule.
5. The check that proves the result.

Do not claim that a feature works unless a command, test, or direct scenario proves it.

## 3. Authority and document order

Use this authority order:

1. `docs/00_SSOT_RULES_AND_SCORING.md`.
2. `docs/01_PRODUCT_ARCHITECTURE.md`.
3. `docs/CODE.md`.
4. `docs/06_SKELETON_PLAN.md`.
5. Other documents in `docs/`.
6. Code comments and README files.

The higher document wins when documents conflict.

Fix the lower document in the same change when the conflict matters.

Do not silently change the SSOT.

A change to the SSOT requires lead approval, a commit that names the rubric reason, and an `EVIDENCE.md` update.

Treat a scope cut as valid when a build gate is at risk.

Treat a new feature as invalid after the feature-freeze gate.

Every scoped work item MUST map to an `EV-*` evidence ID.

Delete work that has no evidence ID unless the work is a necessary defect fix.

## 4. Product boundary

ZeroTrace is an enterprise egress firewall for AI traffic.

The system protects outbound and inbound LLM and agent payloads.

The system runs inside the customer's perimeter.

The system must not store recoverable sensitive values.

The system must not restore original sensitive values.

The main product differentiators are:

- Self-hardening deterministic detection.
- Compositional re-identification risk.
- One-way utility-preserving redaction.
- Agent and tool-result coverage.
- Tamper-evident evidence.

Do not present ordinary entity detection as the main innovation.

Do not add prompt-injection defence to this product without an approved scope change.

Do not turn ZeroTrace into an LLM router, a DSPM system, an endpoint DLP agent, or an identity provider.

## 5. Current implementation state

The current code is Part A of the skeleton plan.

Part A answers which actor group can receive a class of company LLM data.

The current implementation uses FastAPI, SQLAlchemy 2, Alembic, Pydantic, and async database access.

The primary local test dialect is SQLite through `aiosqlite`.

The target development stack is Docker Compose with PostgreSQL, Redis, and the gateway.

The current database model contains these nine tables:

- `tenants`
- `actors`
- `groups`
- `sessions`
- `policies`
- `policy_exceptions`
- `requests`
- `findings`
- `ledger`

`groups` is required because the console must list groups without scanning actor rows.

`Actor` must have an `idp_subject`, a `workload_id`, or both.

The database must not contain `virtual_key_hash`.

`Finding` stores the span path and entity class.

`Finding` must not store the sensitive value.

The policy action order is:

`allow < warn < tokenize < mask < block`.

A business-unit policy may raise an action.

A business-unit policy must not lower an organisation action.

The policy engine rejects a weakening at publish time.

The inbound policy resolves clearance from the actor role and group.

An unknown actor is served as an unregistered actor.

The unregistered actor receives the `unregistered_workload` policy.

The current identity header path is spoofable.

Document this limitation whenever you describe the skeleton identity path.

`zerotrace/detect/stub.py` is a deliberate no-op detector.

The live Part A path reports `detection_stub`.

`zerotrace/gateway/upstream.py` contains the current upstream stub.

The live Part A path reports `upstream_stub`.

`zerotrace/gateway/redact.py` implements `mask` and `block`.

`tokenize` currently degrades to `mask` with an explicit degradation reason.

Do not create a fake token.

`zerotrace/ledger/chain.py` writes and verifies the tenant hash chain.

The ledger must contain classes, paths, decisions, policy versions, and hashes.

The ledger must not contain sensitive values.

The live Part A path is not the full product.

Do not claim that Part B detection, the real upstream, streaming, OIDC, SCIM, the vault, or the interception integrations are complete unless current code and verification prove it.

## 6. Runtime data flow


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Parth-P-Parekh/Zero-Trace-](https://github.com/Parth-P-Parekh/Zero-Trace-) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-30 -->
