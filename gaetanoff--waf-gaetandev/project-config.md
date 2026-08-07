---
trigger: always_on
description: Changelog management, semantic versioning, and release checklist for spec-driven projects
---


# Core Changelog & Release Management

> Every release is a set of spec promotions. The changelog documents what contracts changed, not what code changed.

---

## Semantic Versioning — Spec-Driven Rules

Version increments are driven by the **spec changes**, not by line count or effort.

```
MAJOR.MINOR.PATCH

MAJOR — Breaking contract change (field removed, type changed, endpoint renamed)
MINOR — Additive contract change (new optional field, new endpoint, new event)
PATCH — Non-contract change (bug fix within existing contract, perf, docs)
```

### Version Determination Workflow

```
1. List all spec changes in this release
2. For each changed spec, classify the change (breaking / additive / patch)
3. Apply the highest-severity classification to the release version
4. If any spec is MAJOR → bump MAJOR, reset MINOR and PATCH to 0
5. If highest is MINOR → bump MINOR, reset PATCH to 0
6. If all are PATCH → bump PATCH only
```

### Breaking Change Definition (triggers MAJOR)

A breaking change is any change that **requires existing clients to update**:

- Removing a field from a response schema
- Renaming a field in a request or response
- Changing a field's type (string → number, optional → required)
- Removing or renaming an endpoint
- Changing an HTTP method for an existing endpoint
- Changing authentication scheme
- Removing an enum value that clients may have stored
- Changing event payload structure in AsyncAPI specs

---

## Changelog Format — Keep a Changelog Standard

Every project maintains a `CHANGELOG.md` at the root with this structure:

```markdown
# Changelog

All notable changes to this project will be documented in this file.
Format: [Keep a Changelog](https://keepachangelog.com/en/1.1.0/)
Versioning: [Semantic Versioning](https://semver.org/spec/v2.0.0.html)

## [Unreleased]
<!-- Changes staged for next release. Updated with every merge to main. -->

### Added
- POST /users endpoint (spec: api-user-003, status: validated)

### Changed
- User.email field now requires RFC 5322 format (spec: schema-user-001 v1.3.0)

### Deprecated
- GET /users/search — use GET /users?q= instead. Sunset: 2025-01-01

### Removed
### Fixed
### Security

---

## [2.1.0] — 2024-03-20

### Added
- Order webhook events via AsyncAPI spec (spec: events-order-001)
- Rate limiting headers on all endpoints (X-RateLimit-Limit, X-RateLimit-Remaining)

### Changed
- Order.status now uses enum values instead of free-form string

### Fixed
- Pagination cursor was not URL-safe (spec: api-order-002 patch)

[2.1.0]: https://github.com/org/repo/compare/v2.0.0...v2.1.0
```

### Changelog Sections

| Section | Use When |
|---|---|
| `Added` | New endpoints, fields, features, events — additive changes |
| `Changed` | Modified behavior within existing contract |
| `Deprecated` | Fields or endpoints marked for removal — include sunset date |
| `Removed` | Previously deprecated items now removed |
| `Fixed` | Bug fixes that do not change the contract |
| `Security` | Vulnerability fixes, auth changes, CVE patches |

**Always link to the relevant spec** in changelog entries: `(spec: api-user-003)`.

---

## Release Process

### Pre-Release Gate — Full Checklist

```
Release Readiness Gate

Spec Status
[ ] All specs referenced in this release are at status: validated
[ ] SPEC-INDEX.md is up to date
[ ] No spec debt items classified as Critical or High remain open
[ ] Breaking changes are documented with migration guide

Code Quality
[ ] All conformance tests pass in CI
[ ] Unit test coverage ≥ 80% on business logic
[ ] Integration tests pass
[ ] No open security vulnerabilities (npm audit / cargo audit / etc.)
[ ] No linting errors
[ ] Build is reproducible (lock files committed)

Changelog & Versioning
[ ] CHANGELOG.md [Unreleased] section is complete
[ ] Version bump applied to package manifest (package.json, Cargo.toml, pyproject.toml, etc.)
[ ] Git tag matches version
[ ] [Unreleased] section renamed to [X.Y.Z] — YYYY-MM-DD
[ ] Compare link added at bottom of CHANGELOG.md

Documentation
[ ] README.md reflects current version and usage
[ ] API documentation regenerated from OpenAPI spec
[ ] Migration guide written for any MAJOR version bump

Deployment
[ ] Staging environment validated
[ ] Rollback plan documented
[ ] Feature flags configured (if applicable)
[ ] Database migrations tested (up and down)
[ ] Dependent services notified of breaking changes
```

### Release Commit Convention

```bash
# Version bump commit
git commit -m "chore(release): v2.1.0"

# Tag the release
git tag -a v2.1.0 -m "Release v2.1.0 — Order webhooks, rate limiting"

# Never push without a complete changelog and passing CI
```

---

## Deprecation Protocol

When deprecating a contract (field, endpoint, event):

### Step 1 — Mark in Spec

```yaml
# In OpenAPI spec
/users/search:
  get:
    deprecated: true
    description: |
      DEPRECATED as of v2.1.0. Sunset date: 2025-01-01.
      Use GET /users?q= instead.
      Migration guide: docs/migration/v2.1.0.md
```

### Step 2 — Add Response Header

```http
Deprecation: true
Sunset: Mon, 01 Jan 2025 00:00:00 GMT
Link: <https://api.example.com/docs/migration/v2.1.0>; rel="deprecation"
```

### Step 3 — Changelog Entry

```markdown
### Deprecated

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GaetanOff/WAF-GaetanDev](https://github.com/GaetanOff/WAF-GaetanDev) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
