---
trigger: always_on
description: This file provides repository-specific guidance for AI coding agents working on Heimdall.
---

# AGENTS.md

This file provides repository-specific guidance for AI coding agents working on Heimdall.

The rules in this file take precedence over generic coding-agent conventions when working in this repository.

## Core principles

### Security comes first

**Heimdall is security infrastructure. Every change must be reviewed for security implications, without exception.**

Heimdall participates directly in authentication, authorization, access-control decisions, identity propagation, request processing, and enforcement at trust boundaries.

A defect can result in unauthorized access, authentication bypass, privilege escalation, information disclosure, credential leakage, or incorrect policy enforcement.

Security therefore applies to every kind of change, including:

* feature work;
* bug fixes;
* refactoring;
* performance optimization;
* configuration;
* dependency updates;
* parsing and serialization;
* logging and error handling;
* tests;
* documentation;
* utility code.

Assume externally supplied data is attacker-controlled until trust has explicitly been established.

Preserve:

* trust boundaries;
* authentication and authorization guarantees;
* fail-closed behavior;
* confidentiality of credentials and security-sensitive information;
* isolation of request-specific state.

**Correctness and security take precedence over convenience, abstraction purity, allocation reduction, and performance.**

Never weaken a security control merely to simplify an implementation, make a test pass, or ease an integration.

### Preserve the project's design goals

When making changes, preserve:

* **Security** — maintain security guarantees and trust boundaries.
* **Performance** — avoid unnecessary work on request-processing paths.
* **Clear abstractions** — keep responsibilities and boundaries explicit.
* **Simplicity** — prefer understandable solutions over clever ones.
* **Secure defaults** — failures at security boundaries must not silently become successful decisions.

## Before making changes

Before modifying code:

1. Read the implementation in the affected package.
2. Read the corresponding tests.
3. Inspect adjacent code for established patterns.
4. Identify affected trust boundaries and security assumptions.
5. Consider malformed, ambiguous, unexpected, and malicious input.
6. Check whether related documentation, schemas, configuration, examples, or deployment artifacts are affected.

Prefer extending existing patterns over introducing another way to solve the same problem.

Keep changes focused. Avoid unrelated cleanup and refactoring.

## Repository map

Important areas include:

* `main.go` — application entry point.
* `cmd/` — CLI commands.
* `internal/app/` — application-level contracts and context.
* `internal/config/` — application configuration.
* `internal/pipeline/` — core request-processing abstractions.
* `internal/rules/` — rule matching, construction, repositories, and execution.
* `internal/rules/mechanisms/` — authentication, authorization, contextualization, finalization, and error handling.
* `internal/handler/` — proxy, decision, management, HTTP, and Envoy authorization adapters.
* `internal/keyregistry/` and `internal/secrets/` — key and secret handling.
* `internal/x/` — shared lower-level helpers.
* `schema/` — schemas exposed to users or tooling.
* `charts/heimdall/` — Helm chart and chart tests.
* `docs/content/docs/` — product and reference documentation.
* `docs/content/guides/` — task-oriented guides.
* `docs/openapi/` — OpenAPI documentation.
* `examples/` — example deployments and configurations.
* `examples/docker-compose/` — runnable Docker Compose quickstarts.
* `docker/` — container image definitions.
* `.github/ISSUE_TEMPLATE/` — issue templates.
* `.github/pull-request-template.md` — pull request template.

Security-sensitive behavior is not limited to security-named packages.

## Architecture

### Pipelines

`internal/pipeline` defines fundamental request-processing contracts and carries security-relevant state between steps.

Existing mechanism kinds include:

* authenticator;
* authorizer;
* contextualizer;
* finalizer;
* error handler.

Do not duplicate these concepts merely to avoid using existing abstractions.

Changes to pipeline interfaces are cross-cutting. Inspect implementations, mocks, rules, execution, and handlers before changing them.

Ensure changes cannot:

* lose or overwrite identity information;
* mix state from different requests;
* bypass required pipeline stages;
* incorrectly reuse previous results;
* turn fail-closed behavior into fail-open behavior.

### Rules and mechanisms

Concrete mechanisms belong under the corresponding area in `internal/rules/mechanisms`.

When adding or changing a mechanism:

* follow comparable existing implementations;
* preserve mechanism boundaries;
* reuse shared infrastructure;
* preserve validation and secure defaults;
* consider bypass and malformed-input scenarios;
* add positive, negative, and failure-path tests;
* update schemas, examples, and documentation where necessary.

Authentication and authorization mechanisms must not silently accept malformed, incomplete, ambiguous, or unverifiable input unless that behavior is explicitly part of their contract.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dadrus/heimdall](https://github.com/dadrus/heimdall) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
