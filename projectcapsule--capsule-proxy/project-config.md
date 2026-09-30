---
trigger: always_on
description: These instructions apply throughout this repository. Capsule Proxy's primary focus
---

# Agent instructions for Capsule Proxy

These instructions apply throughout this repository. Capsule Proxy's primary focus
is **speed: acting only as a thin intermediary to the Kubernetes API, with minimal
added latency and minimal deviations from native API behavior**. Its role is to
authenticate, enforce the necessary tenant access boundaries, and forward requests
efficiently. Kubernetes remains responsible for API semantics and resource
lifecycle; Capsule supplies tenant membership and namespace state. Preserve
tenant isolation while minimizing work in the proxy. Extend the existing design
and follow the conventions of the package you are changing.

## Required outcomes

- **Prioritize speed and low overhead.** Minimize added latency, allocations,
  buffering, and API round trips on every request without weakening isolation.
- **Preserve Kubernetes API behavior by default.** Limit request and response
  changes to what secure forwarding and tenant access require; justify and test
  every necessary deviation.
- **Keep the proxy an intermediary.** Delegate native API validation, persistence,
  and resource lifecycle to Kubernetes and tenant reconciliation to Capsule.
- **Implement access features through the existing proxy modules and settings
  APIs.** Extend the relevant request path, `ProxySetting`, or
  `GlobalProxySettings` instead of introducing a parallel permission mechanism.
- **Design around the effective access of each caller.** Identity, tenant
  membership, resource scope, operation, selectors, and reflected RBAC all matter.
- Align every feature with the current repository structure and architecture.
- Search for existing implementations before adding code. Reuse or extend existing
  types, helpers, modules, middleware, controllers, caches, and test fixtures.
- Every code or behavior change requires unit tests and end-to-end (e2e) tests.
  Add or extend coverage for the changed behavior; an unrelated passing suite is
  insufficient. For documentation-only changes, verify paths, commands, and claims
  against the repository instead of adding tests that merely assert text.
- Every e2e change must cover positive and negative cases with one or more real
  Tenant objects present. Add multiple tenants whenever isolation or shared state
  is involved.
- **Run e2e locally only for new feature tests and subsystems/components impacted by
  the change. The full e2e suite runs in GitHub Actions.** Collect observations with
  minimal reasoning during execution; perform forensics after the relevant suites
  have finished.
- New or materially changed performance-sensitive execution paths require benchmark
  coverage; extend existing benchmarks where appropriate.
- **All proxy request-path changes are performance critical**, including changes
  to authentication, authorization, middleware, modules, selectors, lookups,
  caches, response handling, or configuration used by requests.
- **Reuse existing local mutex-protected caches and indexed lookups wherever their
  consistency guarantees fit the operation.** Avoid repeated identity resolution,
  redundant API reviews, and full-list filtering when existing facilities apply.
- A change is not fully validated until its required checks pass. Report missing
  coverage, unavailable environments, and unrun checks explicitly.
- **Every change requires a self-review of scalability, performance, and security.**
  Explain the impact in each area, address findings and feedback, and repeat the
  review and relevant validation until the completion criteria below are met.

## Primary design direction: a fast, minimal Kubernetes API intermediary

Start feature design with the native Kubernetes request and response. Identify the
smallest intervention needed for tenant access, its added cost, and any observable
deviation from the upstream API. Prefer forwarding through the existing path with
minimal work. Establish the caller, resource scope, access policy, and upstream
identity; a successful response alone does not prove the access boundary is correct.

- Keep Kubernetes responsible for its API semantics. Avoid duplicating API-server
  validation, storage, resource lifecycle, or general-purpose policy processing in
  the proxy. Extend proxy settings only to express the access the intermediary needs.
- Preserve upstream request/response behavior outside the necessary access changes.
  Explain why each new rewrite, synthesized response, or interception is required
  and test both its intended effect and unaffected Kubernetes behavior.
- Evaluate the cost of every additional lookup, review, decode/encode, copy, and
  buffer. Choose the least work that preserves authorization and correctness; use
  measurements to justify added work on common paths.
- Start with `internal/modules/`, `internal/webserver/webserver.go`, and
  `internal/tenant/`. Reuse `modules.Module` and the existing registration path for
  intercepted resource operations; keep resource-specific behavior in its module.
- Extend `api/v1beta1/` for declarative access settings. A namespaced
  `ProxySetting` applies to its associated tenant; `GlobalProxySettings` grants
  selected cluster-resource access to matching subjects without requiring tenant

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [projectcapsule/capsule-proxy](https://github.com/projectcapsule/capsule-proxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
