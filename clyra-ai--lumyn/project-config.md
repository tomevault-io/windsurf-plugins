---
trigger: always_on
description: Status: Normative v3.1 planning contract; product runtime not implemented
---

# AGENTS.md — Lumyn Repository Contract

Version: 3.1
Status: Normative v3.1 planning contract; product runtime not implemented
Scope: This repository only.

## 1. Scope And Intent

- Build Lumyn as the provider-to-consumer application layer for consequential
  API and SDK changes: provider-originated intent to consumer-controlled,
  tested draft PR and consented rollout evidence.
- Treat `docs/product/prd.md` as the product source of truth.
- Treat `docs/product/plan.md` as the human-readable active plan.
- Treat the v3.1 operating documents and the compiled control set under
  `.factory/artifacts/prd-to-plan/lumyn-migration-mvp/` as one planning
  authority.
- Do not describe bounded-agent migration execution, patch delivery, branch
  delivery, PR-bundle delivery, or draft-PR delivery as implemented.
- Regenerate the complete active v3.1 control set whenever its PRD, plan,
  acceptance, task, validation, or closure semantics change. Do not hand-edit
  one compiled artifact to bypass another.
- Keep factoryd dispatch paused. The external Factory
  `profiles/lumyn.yaml` profile and the factoryd bundle/runtime have not been
  requalified against this v3.1 generation.
- Treat the M1 public/synthetic corpus and developer-harness implementation
  packet as immutable historical task evidence. M1's benchmark and lifecycle
  scope is closed with recorded process debt, but none of its 15 linked
  product acceptance items is terminal. This rebaseline authorizes no new
  Lumyn product runtime implementation or live product action.
- Keep `.factory/artifacts/prd-to-plan/lumyn-mvp/` and its task, pilot, and
  lifecycle artifacts immutable as historical evidence.
- Keep Factory run evidence under `.factory/artifacts/`, scratch under
  `.factory/tmp/`, and daemon state under `.factoryd/`.
- Keep consumer-private runtime artifacts in an explicitly configured,
  non-committable root outside the consumer checkout and every public source
  repository.
- Keep independent review, holdout, trace-grade, attestation, shipping, and PR
  lifecycle evidence lifecycle-owned. `task-executor` may not write
  `.factory/artifacts/lifecycle-evidence/` or
  `.factory/artifacts/pr-lifecycle/`.

## 2. North Star

Every product change should improve one or more of:

- provider-paid completion of a consequential API or SDK sunset campaign;
- provider-originated, reusable, confirmed change intent;
- revocable consumer installation and event-specific authorization;
- consumer-controlled repository access and execution;
- bounded deterministic or agent-assisted patch candidates;
- deterministic, baseline-aware repository and workflow verification;
- tested draft-PR delivery with legible patch, local-branch, and PR-bundle
  fallback;
- event-bound, consumer-consented provider rollout status;
- exact Agent Runner and model egress, credential, network, disclosure, cost,
  and provenance controls;
- proof-honest residual-risk reporting;
- consumer review and human merge authority;
- fail-closed handling of unsupported or ambiguous integrations.

## 3. Product Authorities: Two Principals, Two Authorities

Keep the two principals separate:

- The API Provider owns API/SDK intent, the sunset objective, compatibility
  window, supported semantics, and campaign sponsorship.
- The API Consumer Organization owns repository access, commands, model egress,
  credentials, execution, disclosure, branch policy, review, and merge.

Provider payment or campaign sponsorship never grants consumer repository
authority. Consumer participation never lets Lumyn invent or rewrite provider
intent.

Use explicit terms:

- `api_provider` or `change_authority` for the API seller;
- `agent_runner_vendor` for the company supplying the selected coding-agent
  harness;
- `model_provider` for the endpoint used by bounded-agent execution;
- `api_consumer_organization` for the repository-owning organization;
- `consumer_maintainer` for the human with approval and merge authority;
- `lumyn_operator` for the service operator coordinating the campaign.

Do not use bare `provider` where the meaning could be ambiguous.

## 4. Non-Negotiable Product Constraints

- Analyze only explicitly authorized repositories.
- Never claim coverage of all downstream integrations.
- A Provider Change Contract is authoritative when accountably confirmed. Its
  provider event and contract remain non-executable data, and v3.1 does not
  require an elaborate PKI, universal event network, or receipt protocol.
- Provider material is data, never executable authority. Do not execute
  provider-supplied scripts or let repository/provider content widen tools,
  permissions, network access, or writable paths.
- Record stable change identity, audience, source, target, semantic intent,
  unresolved questions, provenance, confirmation, and
  supersession/withdrawal state used by every migration plan.
- The first provider channel is a signed versioned manifest at a pinned
  provider-controlled HTTPS URL. It embeds the Provider Change Contract or
  pins its exact provider-controlled URL. Verify origin, enrolled key,
  sequence, freshness, retrieved-byte contract digest, audience, and lifecycle
  state; attended import is recovery and cannot authorize
  installed-preauthorization writes.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Clyra-AI/lumyn](https://github.com/Clyra-AI/lumyn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
