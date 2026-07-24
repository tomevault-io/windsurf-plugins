---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

🔹 **What Flamingock IS**

Flamingock is a platform for the audited, synchronized evolution of distributed systems.

It enables Change-as-Code (CaC): all changes to external systems (schemas, configs, storage, infra-adjacent systems, etc.) are written as versioned, executable, auditable units of code.

It applies changes safely, in lockstep with the application lifecycle, not through CI/CD pipelines.

It provides a Client Library (open-source, Community Edition) and a Cloud Backend (SaaS or Self-Hosted) for governance, visibility, and advanced features.

It works across databases (MongoDB, DynamoDB, SQL, etc.), event schemas (Kafka + Schema Registry, Avro, Protobuf), configs, S3 buckets, queues, and more.

It ensures auditability, safety, synchronization, governance, and visibility across all system evolutions.

🔹 **What Flamingock is NOT**

It is not a database migration tool tied to a single DB (like Mongock, Flyway, or Liquibase).

It is not a CI/CD pipeline or a replacement for tools like GitHub Actions, Jenkins, or ArgoCD.

It is not an infra-as-code tool like Terraform or Pulumi (though it's conceptually close in ambition, but focused on system evolution instead of infra provisioning).

It is not limited to databases — databases are only one type of target system.

🔹 **Goals of Flamingock**

Unify external system evolution under a single, auditable, code-driven model.

Ensure safety and resilience with strong execution guarantees (idempotency, manual intervention, and safe retry in Cloud).

Provide governance & compliance via audit logs, approvals, visibility, and policy controls.

Boost developer productivity by making changes versioned, testable, and executable in sync with the app lifecycle.

Enable organizational coordination for distributed teams and services evolving multiple systems in parallel.

🔹 **Ambitions & Vision**

Become the standard for controlled, auditable, and intelligent system evolution, in the same way Terraform became the standard for infrastructure.

Extend Change-as-Code (CaC) to all external dependencies of an application (schemas, configs, storages, event systems, etc.).

Provide a cloud-native platform (Cloud Edition) with governance, dashboards, approvals, observability, and AI-assisted evolution planning.

Build an open-core business model:
- Community Edition → OSS, self-contained, no backend.
- Cloud Edition → SaaS, premium automation and governance features.
- Self-Hosted Edition → same as Cloud, but deployable on customer infra.

**👉 North Star:** Flamingock = Change-as-Code platform for audited, synchronized evolution of distributed systems. Not just DB migrations. Not CI/CD. Not infra-as-code. Its ambition = Terraform-equivalent for system evolution.

## Terminology Guidelines

When writing code, documentation, or user-facing content for Flamingock:

### DO NOT use "migration" or "migrations"
- Flamingock is about **external system evolution**, not just database migrations
- Use "**changes**" (for individual units) or "**system evolution**" (for the concept)
- Example: "Apply pending changes" NOT "Apply pending migrations"

### CLI Naming
- The `cli/flamingock-cli-executor` module is THE Flamingock CLI (temporary internal name during transition)
- User-facing content should say "**Flamingock CLI**", not "CLI Executor"
- The old `cli/flamingock-cli` module is legacy and will be deprecated once feature parity is achieved

### Framework-Agnostic Messaging
- The CLI is designed to be framework-agnostic
- Currently only Spring Boot is supported, but this will expand
- User-facing CLI content should NOT mention "Spring Boot" - just say "applications"

## Build System

This is a multi-module Gradle project using Kotlin DSL.

### Common Commands

```bash
# Build entire project
./gradlew build

# Run tests for entire project
./gradlew test

# Run tests for specific module
./gradlew :core:flamingock-core:test

# Build and publish locally
./gradlew publishToMavenLocal

# Clean build
./gradlew clean build

# Run tests with debugging info
./gradlew test --info

# Build specific module only
./gradlew :core:flamingock-core:build
```

### Release Commands

```bash
# Release specific module
./gradlew -Pmodule=flamingock-core jreleaserFullRelease

# Release entire bundle
./gradlew -PreleaseBundle=core jreleaserFullRelease
./gradlew -PreleaseBundle=community jreleaserFullRelease
./gradlew -PreleaseBundle=cloud jreleaserFullRelease
```

## Architecture Overview

### Core Components

**Flamingock Builder Pattern**: Central configuration through `AbstractFlamingockBuilder` with hierarchical builder inheritance:
- `CommunityFlamingockBuilder` - Community Edition
- `CloudFlamingockBuilder` - Cloud Edition  
- `AbstractFlamingockBuilder.build()` method orchestrates component assembly with critical ordering dependencies

**Context System**: Hierarchical dependency injection via `Context` and `ContextResolver`:
- Base context contains runner ID and core configuration
- Plugin contexts merged via `PriorityContextResolver`
- External frameworks (Spring Boot) contribute dependency contexts

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [flamingock/flamingock-java](https://github.com/flamingock/flamingock-java) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
