---
trigger: always_on
description: This file provides guidance to AI coding agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI coding agents when working with code in this repository.

## Mission And Scope

OngoingAI Gateway is a transparent, auditable proxy between applications and AI providers. It should enforce access control and capture trustworthy trace/usage data while staying invisible to client workflows.

- Security first: no credential leakage, strong tenant isolation, least-privilege access, auditable behavior.
- Reliability first: streaming integrity, predictable failure modes, and stable operation under load.
- Practical developer experience: fast local setup, clear config, and easy-to-extend provider integrations.
- Scope boundary: this repo is a headless AI gateway and owns gateway M2M auth/enforcement; UI/user-session concerns are out of scope.

Design intent:
- Self-hosted default: single binary, SQLite by default.
- Team/multi-instance mode: Postgres-backed storage and dynamic key management.
- Upstream provider API keys are pass-through only and must never be persisted.
- Gateway keys are identity/authorization credentials for the proxy and are distinct from provider credentials.

## High-Priority Outcomes

When implementing features or fixes, optimize for these outcomes in order:

1. Prevent security regressions.
2. Preserve correctness and tenant boundaries.
3. Preserve low-latency proxy behavior.
4. Keep architecture simple and maintainable.
5. Improve observability and operator experience.

## Go Engineering Standards

- Write idiomatic Go. Keep functions small, cohesive, and explicit.
- Keep interfaces at boundaries (`trace`, `configstore`, providers); avoid speculative abstractions.
- Pass `context.Context` as the first parameter for request-bound or storage calls.
- Wrap errors with actionable context (`fmt.Errorf("load gateway key: %w", err)`).
- Use deterministic behavior for critical logic (auth, limits, tenant scoping, redaction policies).
- Avoid global mutable state. Prefer dependency injection via constructors.
- Keep hot-path allocations low in middleware/proxy code.
- Protect concurrency edges: no data races, leaked goroutines, or unbounded queues.
- Use structured logs and never log secrets, tokens, raw credentials, or unredacted sensitive payloads.
- Add comments where helpful, especially for non-obvious "why" decisions. Do not add redundant comments.
- Keep exported symbols documented with concise Go doc comments.

## Error Handling Conventions

- Return errors when callers can recover, retry, or map to clear HTTP responses.
- In proxy request paths, fail closed for authn/authz, tenant resolution, and policy enforcement errors.
- For non-critical telemetry failures (for example trace enqueue/write issues), log structured errors, emit metrics, and continue serving proxy traffic.
- Startup/config/store initialization failures should fail fast with a non-zero exit.
- Never panic for expected runtime conditions or user input errors; reserve panic for programmer bugs that cannot be safely recovered.

## Security Requirements

- Enforce fail-closed behavior for auth/key-store uncertainty on proxy access decisions.
- Preserve the pass-through credential model: provider API keys are forwarded to upstream providers but never persisted locally.
- Never store upstream provider API key material in config store, trace store, logs, or errors.
- Maintain strict `org_id`/`workspace_id` scoping in every query and access path.
- Validate and sanitize all externally controlled inputs (headers, params, config values).
- Treat PII controls and redaction policy behavior as security-sensitive code paths.
- Preserve auditability for key lifecycle and authorization decisions.

## Reliability Requirements

- Proxy path must prioritize request forwarding over trace persistence latency.
- Streaming responses must preserve ordering and chunk delivery semantics.
- Async trace pipeline must handle backpressure predictably (drop/queue behavior must be explicit and observable).
- Shutdown paths must avoid silent data loss when feasible (flush and report).
- Provider failures and storage failures should return clear, stable error semantics.

## Testing Standards

Every functional change should include or update tests. Bias toward behavior tests over implementation tests.

- Use `t.Parallel()` where safe.
- Use table-driven tests for auth/policy/validation rules.
- Use descriptive subtest names in table-driven suites (`name: "rejects revoked key"`).
- Use `t.TempDir()` and real SQLite/Postgres-backed paths for storage tests where possible.
- Prefer real integration behavior over extensive mocking in critical paths.
- Add regression tests for every bug fix.
- Name tests descriptively (`TestDynamicMiddlewareRejectsRevokedKey`, not `TestAuth2`).
- Test run patterns:
  - Fast focused run: `go test ./path/to/pkg -run TestName -count=1`
  - Package run: `go test ./internal/auth/`
  - Full run: `make test`
  - Concurrency-sensitive paths before merge: `go test -race ./...`

Minimum test focus areas:

- Security:
  - Missing/invalid/revoked gateway keys.
  - Missing upstream provider credentials.
  - Tenant isolation and cross-tenant access denial.
  - Permission and limit enforcement bypass attempts.
  - Sensitive data handling and redaction behavior.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ongoingai/gateway](https://github.com/ongoingai/gateway) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
