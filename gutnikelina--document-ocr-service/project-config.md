---
trigger: always_on
description: This file is the repository-level operating contract for coding agents. It applies to the entire repository unless a
---

# Codex Engineering Guide

## Purpose and Scope

This file is the repository-level operating contract for coding agents. It applies to the entire repository unless a
more specific `AGENTS.md` exists below the file being changed.

Build a production-grade, high-throughput document-processing service without weakening correctness, security, or
operability. Prefer small, evidence-based changes over speculative abstractions.

## Sources of Truth

Before changing behaviour, read the relevant code and tests, then consult:

1. `docs/requirements/document-ocr-service-functional-spec.md` for business behaviour and NFRs.
2. `docs/architecture/document-ocr-service-tech-spec.md` for architectural intent.
3. `README.md` for the service overview and operational model.
4. `pom.xml` and CI workflows for the actually configured toolchain.

Do not conceal contradictions between documents and implementation. Preserve current behaviour unless the task explicitly
authorises a change, and report material ambiguities rather than choosing a business rule silently.

Key constraints include files up to 50 MB, protected document data, durable asynchronous processing, at-least-once
delivery, and a target end-to-end processing latency of p95 <= 3 seconds per page. Treat the latency target as something
to measure under a documented workload, not as a claim inferred from unit tests.

## Working Agreement

- For review, explanation, diagnosis, or planning requests, inspect and report; do not edit unless asked.
- For build, change, or fix requests, make the smallest complete in-scope change and run relevant non-destructive
  checks.
- Inspect `git status` before editing. Preserve user-owned and unrelated changes, including staged files.
- Ask before destructive operations, external writes, expensive actions, or material scope expansion. Safe local reads,
  edits, builds, and tests do not need confirmation.
- Never use destructive Git commands to discard work. Do not commit, push, or open a pull request unless requested.
- Use `rg`/`rg --files` for discovery. Read the implementation and nearby tests before proposing a pattern.
- Do not add a dependency, infrastructure component, public endpoint, or configuration knob without a concrete need.
- When changing a dependency or framework API, verify compatibility from primary documentation and record the reason.
- Keep the final handoff concise: summarise behaviour changed, checks run, and remaining risks or unverified assumptions.

## Architecture and Boundaries

Use a conventional layered Spring MVC architecture under the service root. Use the following package structure; add a
new peer package only when the codebase has a concrete need that cannot be represented by one of these packages:

```text
config
entity
repository
service
exception
rest
├── api
├── controller
├── dto
└── exception
```

- **config**: Spring dependency wiring, typed configuration properties, external-client setup, executor configuration,
  and startup validation;
- **entity**: persistence entities and cohesive domain state, value types, invariants, and transition rules;
- **repository**: persistence access and bounded queries over entities;
- **service**: business use cases, workflow orchestration, transaction boundaries, integration orchestration, and
  business decisions;
- **exception**: application and domain exceptions that are independent of HTTP and other transports;
- **rest**: namespace for the HTTP adapter; do not place controllers or DTOs directly in the package root;
- **rest.api**: interfaces generated from, or kept aligned with, the OpenAPI contract;
- **rest.controller**: thin Spring MVC controllers that implement `rest.api` interfaces, validate transport input,
  delegate to one service use case, and explicitly map between DTOs and application/entity types;
- **rest.dto**: REST request and response payloads generated from, or kept aligned with, the OpenAPI contract; keep
  them separate from persistence entities and provider-specific models;
- **rest.exception**: HTTP exception mapping, `@ControllerAdvice`, and RFC 9457/`ProblemDetail` response construction.

The primary request flow is `rest.controller -> service -> repository`, with repositories persisting `entity` types.
The `config` package wires dependencies, `exception` represents transport-independent failures, and `rest.exception`
maps those failures to the HTTP contract. Controllers, repositories, configuration classes, and provider SDK
integrations must not contain business decisions. Entity types must not depend on Spring MVC, REST DTOs, Kafka, S3, or
AI provider types. Do not expose JPA entities or provider-specific models through application or API boundaries. Keep
mappings explicit; MapStruct is appropriate when it reduces repetitive mapping without hiding business logic.

In MVC terms, `entity`, `repository`, and `service` form the Model; `rest.controller` is the Controller; and `rest.dto`
is the external REST representation returned in place of a server-rendered View. The `rest.api` package defines the
contract implemented by controllers. Supporting packages must preserve these boundaries rather than bypass them.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GutnikElina/document-ocr-service](https://github.com/GutnikElina/document-ocr-service) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-27 -->
