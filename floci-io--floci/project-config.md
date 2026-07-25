---
trigger: always_on
description: Guidance for AI coding agents working in the Floci repository.
---

Guidance for AI coding agents working in the Floci repository.

This file defines repository-specific operating rules for autonomous or semi-autonomous coding agents. Follow these instructions unless a maintainer explicitly tells you otherwise.

---

## Project Overview

Floci is a Java-based local AWS emulator built on Quarkus.

Its goal is full AWS SDK and AWS CLI compatibility through real AWS wire protocols, not convenience APIs or simplified abstractions.

Floci acts as an open-source alternative to LocalStack Community.

- Port: 4566
- Stack:
  - Java 25
  - Quarkus 3.32.3
  - JUnit 5
  - RestAssured
  - Jackson
  - Docker integrations for Lambda, RDS, and ElastiCache

---

## First Principles

When making changes, follow these priorities:

1. Preserve AWS protocol compatibility
2. Match AWS SDK and CLI behavior
3. Reuse existing Floci patterns
4. Prefer correctness over convenience
5. Keep changes narrow and testable

Critical rules:

- Do not introduce custom endpoint shapes
- Do not change request or response formats for convenience
- Do not perform broad refactors unless the task explicitly requires them
- Keep behavior aligned with AWS expectations and existing Floci conventions

---

## Architecture

Floci follows a layered design:

- **Controller / Handler**
  - Parses AWS protocol input
  - Produces AWS-compatible responses

- **Service**
  - Contains business logic
  - Throws `AwsException`

- **Model**
  - Domain objects

### Core Infrastructure

- `EmulatorConfig`
- `ServiceRegistry`
- `StorageBackend` + `StorageFactory`
- `AwsJson11Controller`
- `AwsQueryController`
- `AwsException` + `AwsExceptionMapper`
- `EmulatorLifecycle`

---

## Package Layout

- `io.github.hectorvent.floci.config`
- `io.github.hectorvent.floci.core.common`
- `io.github.hectorvent.floci.core.storage`
- `io.github.hectorvent.floci.lifecycle`
- `io.github.hectorvent.floci.services.<service>`

Typical service structure:

- `services/<svc>/`
  - `*Controller.java`
  - `*Service.java`
  - `model/`

Rule:
Copy an existing service pattern before introducing a new one.

---

## AWS Protocol Rules

Floci must implement real AWS wire protocols.

| Protocol | Services | Request Format | Response Format | Implementation |
|----------|----------|----------------|-----------------|----------------|
| Query | SQS, SNS, IAM, STS, RDS, ElastiCache, CloudFormation, CloudWatch Metrics | form-encoded POST + `Action` | XML | `AwsQueryController` |
| JSON 1.1 | SSM, EventBridge, CloudWatch Logs, Kinesis, KMS, Cognito, Secrets Manager, ACM | POST + `X-Amz-Target` | JSON | `AwsJson11Controller` |
| REST JSON | Lambda, API Gateway, SES V2 | REST paths | JSON | JAX-RS |
| REST XML | S3 | REST paths | XML | JAX-RS |
| TCP | ElastiCache, RDS | raw protocol | native | proxies |

### Important exceptions

- CloudWatch Metrics supports both Query and JSON 1.1; handlers must remain aligned
- SQS and SNS may expose multiple compatibility paths; do not let them drift
- Cognito well-known endpoints are OIDC REST JSON endpoints, not AWS management APIs
- Data-plane protocols may use raw TCP sockets
- Management APIs should be validated with AWS SDK clients, not only handcrafted HTTP requests

---

## XML / JSON Rules

- Use `XmlBuilder` for XML responses
- Use `XmlParser` for XML parsing; do not use regex
- Use `AwsNamespaces` constants
- JSON errors must follow AWS error structures
- Types returned directly from controllers must remain compatible with native-image reflection requirements

---

## Storage Rules

Supported storage modes:

- `memory`
- `persistent`
- `hybrid`
- `wal`

Rules:

- Always use `StorageFactory`
- Do not instantiate storage implementations directly inside services
- Respect lifecycle hooks for load and flush behavior

Important nuance:

Configuration interfaces may declare fallback defaults, but `application.yml` defines effective runtime behavior. Treat repository YAML as the source of truth unless a task explicitly changes configuration semantics.

When adding storage-related behavior:

1. Update `EmulatorConfig`
2. Update main `application.yml`
3. Update test `application.yml`
4. Wire through `StorageFactory`
5. Verify lifecycle integration

---

## Configuration Rules

Configuration lives under `floci.*`.

When adding config:

1. Add it to `EmulatorConfig`
2. Add it to main `application.yml`
3. Add it to test `application.yml` if needed
4. Update documentation if user-facing
5. Follow `FLOCI_*` environment variable conventions

Critical areas:

- `base-url`
- `hostname`
- region and account defaults
- port ranges
- persistence paths
- Docker networking

---

## Build & Run

    ./mvnw quarkus:dev
    ./mvnw test
    ./mvnw clean package
    ./mvnw clean package -DskipTests

### Focused tests

    ./mvnw test -Dtest=SsmIntegrationTest
    ./mvnw test -Dtest=SsmIntegrationTest#putParameter

---

## Compatibility Project

Compatibility test suite: `./compatibility-tests/`

Guidelines:

- Prefer AWS SDK clients over raw HTTP for management-plane validation
- Use this suite when changes may affect real SDK behavior

---

## Testing Rules

### Conventions

- Unit tests: `*ServiceTest.java`
- Integration tests: `*IntegrationTest.java`
- Prefer package-private constructors for testability

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [floci-io/floci](https://github.com/floci-io/floci) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
