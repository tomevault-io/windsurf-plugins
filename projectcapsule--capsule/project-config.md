---
trigger: always_on
description: These instructions apply throughout this repository. Capsule's central design goal
---

# Agent instructions for Capsule

These instructions apply throughout this repository. Capsule's central design goal
is **namespace profiling through the rules API**: declaratively defining the
policies, permissions, metadata, and resource behavior that apply to each namespace.
Tenants provide ownership, isolation, and a scope for distributing those rules.
Every feature must serve this namespace-profiling goal. Tenant isolation, API
compatibility, and admission latency remain core requirements. Extend the existing
design and follow the conventions of the package you are changing.

## Required outcomes

- **Implement new features primarily through the rules API.** Extend the existing
  rule model and its consumers as the default approach to namespace behavior.
- **Design around the effective profile of each namespace.** Tenant membership
  establishes scope; namespaces within one tenant can require different profiles.
- Align every feature with the current repository structure and architecture.
- Search for existing implementations before adding code. Reuse or extend existing
  types, helpers, handlers, controllers, caches, and test fixtures wherever possible.
- Every change requires unit tests and end-to-end (e2e) tests. Add or extend coverage
  for the changed behavior; an unrelated passing suite is insufficient.
- Every e2e change must cover positive and negative cases with one or more real
  Tenant objects present. Add multiple tenants whenever isolation or shared state
  is involved.
- **Run e2e locally only for new feature tests and subsystems/components impacted by
  the change. The full e2e suite runs in GitHub Actions.** Collect observations with
  minimal reasoning during execution; perform forensics after the relevant suites
  have finished.
- New or materially changed performance-sensitive execution paths require benchmark
  coverage; extend existing benchmarks where appropriate.
- **All admission changes are performance critical**, including changes to shared
  helpers, configuration, rules, lookups, caches, or registration used by admission.
- **Reuse existing local mutex-protected caches and indexed lookups wherever their
  consistency guarantees fit the operation.** Admission must avoid repeated
  compilation, redundant reads, and full-list filtering when these facilities apply.
- A change is not fully validated until its required checks pass. Report missing
  coverage, unavailable environments, and unrun checks explicitly.
- **Every change requires a self-review of scalability, performance, and security.**
  Explain the impact in each area, address findings and feedback, and repeat the
  review and relevant validation until the completion criteria below are met.

## Primary design direction: namespace profiling through rules

Namespace profiling means composing the applicable rules into the desired behavior
of a namespace and the resources within it. Start feature design by identifying the
namespace behavior to express, how namespaces are selected, how rules compose, and
how that behavior is reconciled or enforced. Ownership and tenant lifecycle support
this goal; operating on a Tenant object alone does not establish that a feature is
designed at the right level.

- Start with `pkg/api/rules/`, especially `NamespaceRuleBodyNamespace` and
  `NamespaceRuleBodyTenant`. Keep reusable namespace behavior in the namespace rule
  body, with tenant distribution and namespace selection in the existing wrapper.
- Prefer extending the rules exposed through `Tenant.spec.rules` over adding
  standalone Tenant policy fields, tenant-wide switches, or special-case handlers.
  Existing legacy Tenant fields are compatibility surfaces, not the default model
  for new features. Preserve their behavior when maintaining them.
- Reuse namespace selection, rule ordering, audience filtering, templating, and
  effective-rule evaluation. Do not assume every namespace in a tenant has the
  same profile or reduce rule composition to a tenant-wide boolean.
- Follow the relevant existing path through `pkg/tenant/rules.go`, tenant and
  `rulestatus` controllers, `RuleStatus`, `pkg/ruleengine/`, and
  `internal/webhook/rules/`. `RuleStatus.status.rules` holds effective namespace
  enforcement rules; preserve its ordering and the established fallback behavior.
  Other rule effects, such as quota generation and permissions, must extend their
  existing reconciliation paths rather than being forced into enforcement status.
- A feature implemented outside the rules API must explain why the rule model is
  unsuitable and how the feature still supports namespace profiling. Infrastructure
  and tenant lifecycle changes should enable that model without adding a parallel
  policy mechanism. This direction does not authorize unrelated API migrations or
  a new NamespaceProfile resource.
- Demonstrate the resulting namespace behavior in tests. For rules features, cover
  selected and non-selected namespaces, different profiles within one tenant,
  composition with existing rules, and changes to rules or namespace labels. Keep
  the required positive/negative cases and cross-tenant isolation coverage.

## Start with the repository


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [projectcapsule/capsule](https://github.com/projectcapsule/capsule) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
