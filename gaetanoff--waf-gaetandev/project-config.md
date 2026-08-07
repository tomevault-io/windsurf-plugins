---
trigger: always_on
description: Conformance gates — automated and manual validation that code exactly matches specifications, with PR checklist and hard blocking rules
---


# Quality Gates (Spec-Driven)

> A quality gate is not "do the tests pass?" It is "does the system exactly match the contract?" These gates are non-negotiable. No exceptions for "it works locally" or "the client can adapt."

---

## Gate Philosophy

Gates are either **blocking** (merge is rejected until resolved) or **releasing** (deployment is blocked until resolved). There are no advisory gates — every issue is either fixed or explicitly documented as accepted risk with a timeline.

---

## Gate 1 — Spec Validity (Blocking)

Before any code is considered, specs must be syntactically and semantically valid.

### Checks

| Check | Tool | Pass Condition |
|---|---|---|
| OpenAPI 3.1 validity | `spectral lint` | 0 errors, 0 warnings |
| JSON Schema validity | `ajv compile` | 0 errors |
| Gherkin syntax | `cucumber --dry-run` | 0 parse errors |
| AsyncAPI validity | `spectral lint` (AsyncAPI ruleset) | 0 errors |
| All `$ref` resolvable | `redocly lint` | 0 unresolved refs |

### Enforcement
```bash
# CI step
npm run spec:lint
# Must exit 0. Any error blocks the pipeline.
```

### Fail → Fix Protocol
A spec lint failure means the spec is incomplete or malformed. Fix the spec — never bypass this gate.

---

## Gate 2 — Code Generation (Blocking)

Types and interfaces generated from specs must compile without errors.

### Checks
- TypeScript types generated from JSON Schema compile: `tsc --noEmit`
- OpenAPI client/server stubs compile (if using generator)
- No type drift between spec and implementation (no `as any` escape hatches)

### Enforcement
```bash
npm run spec:generate && npm run typecheck
```

If generation fails, the spec is invalid or incompatible with the target. Fix the spec, not the generator output.

---

## Gate 3 — API Conformance (Blocking)

The running application must respond exactly as the OpenAPI spec defines.

### What "Exactly" Means
- Correct HTTP status codes (201 not 200 for resource creation)
- Response body matches JSON Schema referenced in the spec (no extra undocumented fields, no missing required fields)
- Request validation rejects invalid inputs with the correct error codes
- Auth requirements enforced as declared in `securitySchemes`
- Headers present as required by spec (Content-Type, Location, ETag, etc.)

### Tools

| Tool | Use Case |
|---|---|
| Dredd | Run OpenAPI spec as a test suite against live server |
| Prism | Mock + validation proxy; contract testing |
| Portman | Generate Postman/Newman tests from OpenAPI |
| Pact | Consumer-driven contract testing for microservices |
| Schemathesis | Property-based API testing from OpenAPI |

```bash
# Start server
npm run start:test &

# Run conformance suite
npx dredd specs/api/openapi.yaml http://localhost:3000

# Or with Prism in validation mode
npx prism proxy specs/api/openapi.yaml http://localhost:3000 --errors
```

### Hard Rules
- Every endpoint in the OpenAPI spec must have at least one passing conformance test
- Undocumented endpoints (routes in code with no OpenAPI entry) fail the gate
- Response body with extra undocumented fields fails the gate (use `additionalProperties: false`)

---

## Gate 4 — Behavior Conformance (Blocking)

All Gherkin scenarios in `specs/features/` must pass.

### Scope
- Happy path scenarios (required)
- Error path scenarios (required for all `4xx` and `5xx` codes in spec)
- Edge cases (empty inputs, boundary values, concurrency scenarios)

```bash
npx cucumber-js specs/features/ --require test/steps/
```

### Coverage Target
- 100% of `specs/features/` scenarios must have step definitions and pass
- No skipped (`@wip`) scenarios in a validated spec — they indicate incomplete implementation

---

## Gate 5 — Security (Blocking)

### Automated Checks
| Check | Tool | Pass Condition |
|---|---|---|
| No hardcoded secrets | `trufflehog` / `gitleaks` | 0 detections |
| Dependency vulnerabilities | `npm audit --audit-level=high` | 0 high/critical |
| SAST analysis | `semgrep` | 0 high severity findings |
| Auth enforcement | Custom conformance test | All protected endpoints return 401 without token |

### Manual Check
- Auth scheme in code matches `securitySchemes` in OpenAPI spec
- Role-based access control matches actor permissions defined in discovery
- PII fields not logged (cross-reference logging config with data schemas)

```bash
npm run security:audit
# Must exit 0 for merge.
```

---

## Gate 6 — Performance (Release-Blocking)

Performance gates block release, not merge. Measured against SLOs in `specs/slos/*.slo.yaml`.

### Default SLO Thresholds (override in your SLO spec)

| Endpoint Type | p50 | p95 | p99 | Error Rate |
|---|---|---|---|---|
| Read (GET) | < 50ms | < 200ms | < 500ms | < 0.1% |
| Write (POST/PUT/PATCH) | < 100ms | < 500ms | < 1s | < 0.5% |
| Heavy computation | < 500ms | < 2s | < 5s | < 1% |
| Async job trigger | < 50ms | < 200ms | < 500ms | < 0.1% |

### Tools
- `k6` for load testing
- `autocannon` for HTTP benchmarking
- `clinic.js` for Node.js profiling

---

## Gate 7 — PR Checklist (Blocking — Human Review)

Every pull request must include a signed-off checklist. No merge without all boxes checked.

```markdown
## PR Conformance Checklist

### Spec

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GaetanOff/WAF-GaetanDev](https://github.com/GaetanOff/WAF-GaetanDev) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
