---
trigger: always_on
description: Guidance for AI coding agents working in the floci-gcp repository.
---

Guidance for AI coding agents working in the floci-gcp repository.

This file defines repository-specific operating rules for autonomous or semi-autonomous coding agents. Follow these instructions unless a maintainer explicitly tells you otherwise.

---

## Project Overview

floci-gcp is a Java-based local GCP emulator built on Quarkus.

Its goal is full GCP SDK and gcloud CLI compatibility through real GCP wire protocols, not convenience APIs or simplified abstractions.

floci-gcp acts as an open-source alternative to the GCP-provided emulators, unified under a single port.

- Port: 4588
- Stack:
  - Java 25
  - Quarkus 3.34.6
  - JUnit 5
  - RestAssured
  - Jackson
  - quarkus-grpc (gRPC + HTTP/2 via ALPN on the same port)

---

## First Principles

When making changes, follow these priorities:

1. Preserve GCP protocol compatibility
2. Match GCP SDK and gcloud CLI behavior
3. Reuse existing floci-gcp patterns
4. Prefer correctness over convenience
5. Keep changes narrow and testable

Critical rules:

- Do not introduce custom endpoint shapes
- Do not change request or response formats for convenience
- Do not perform broad refactors unless the task explicitly requires them
- Keep behavior aligned with GCP expectations and existing floci-gcp conventions

---

## Architecture

floci-gcp follows a layered design:

- **Controller / Handler**
  - Parses GCP protocol input (gRPC or REST)
  - Produces GCP-compatible responses

- **Service**
  - Contains business logic
  - Throws `GcpException`

- **Model**
  - Domain objects

### Core Infrastructure

- `EmulatorConfig` — `@ConfigMapping(prefix = "floci-gcp")` SmallRye Config interface
- `ServiceRegistry`
- `StorageBackend` + `StorageFactory`
- `GcpException` + `GcpExceptionMapper`
- `GcpGrpcController` — base class for gRPC service implementations
- `ProjectContextFilter` — extracts GCP project ID from request path or headers
- `RequestContext` — `@RequestScoped` holder for the current project ID
- `GcpResourceNames` — utilities for parsing and building GCP resource name strings
- `EmulatorLifecycle`
- `XmlBuilder` + `XmlParser` — used by GCS (REST XML)

---

## Package Layout

- `io.floci.gcp.config`
- `io.floci.gcp.core.common`
- `io.floci.gcp.core.common.dns`
- `io.floci.gcp.core.common.docker`
- `io.floci.gcp.core.storage`
- `io.floci.gcp.lifecycle`
- `io.floci.gcp.lifecycle.inithook`
- `io.floci.gcp.services.<service>`

Typical service structure:

- `services/<svc>/`
  - `*Controller.java`
  - `*Service.java`
  - `model/`

Rule:
Copy an existing service pattern before introducing a new one.

---

## GCP Protocol Rules

floci-gcp must implement real GCP wire protocols.

| Protocol | Services | Transport | Implementation |
|----------|----------|-----------|----------------|
| gRPC | Pub/Sub, Firestore, Datastore, Secret Manager | HTTP/2 + proto3 | `GcpGrpcController` subclass |
| REST JSON | GCS (management), IAM, Secret Manager (REST) | HTTP/1.1 or HTTP/2 | JAX-RS |
| REST XML | GCS (object operations) | HTTP/1.1 or HTTP/2 | JAX-RS + `XmlBuilder` |

### Single-port design

Both gRPC and REST are served on port **4588** via ALPN negotiation:
- `quarkus.http.http2=true`
- `quarkus.grpc.server.use-separate-server=false`

### Auth bypass

GCP SDKs skip credential checks when `*_EMULATOR_HOST` environment variables are set. floci-gcp does not validate credentials; it accepts all requests unconditionally.

### Project ID as multi-tenancy key

GCP resource names follow `projects/{project}/...`. The project ID is the multi-tenancy boundary. All storage keys are namespaced by project ID via `ProjectAwareStorageBackend`.

Resolution order in `ProjectContextFilter`:
1. URL path segment `projects/{project}/...`
2. `x-goog-request-params` header (`project=...`)
3. `EmulatorConfig.defaultProjectId()` fallback

### Important exceptions

- GCS uses REST XML for object operations and REST JSON for bucket management; keep them aligned
- gRPC services use pre-compiled stubs from `grpc-google-cloud-*-java` artifacts — do not introduce raw `.proto` codegen
- Management APIs should be validated with GCP SDK clients, not only handcrafted HTTP requests

---

## XML / JSON Rules

- Use `XmlBuilder` for XML responses (GCS object API)
- Use `XmlParser` for XML parsing; do not use regex
- JSON errors must follow GCP error structures: `{"error": {"code": 404, "message": "...", "status": "NOT_FOUND"}}`
- gRPC errors must map to `io.grpc.Status` codes via `GcpException.grpcCode()`
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
- Storage keys are namespaced by GCP project ID via `ProjectAwareStorageBackend`

Important nuance:

`EmulatorConfig` declares `@WithDefault` values, but `application.yml` defines effective runtime behavior. Treat repository YAML as the source of truth unless a task explicitly changes configuration semantics.

When adding storage-related behavior:

1. Update `EmulatorConfig`
2. Update main `application.yml`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [floci-io/floci-gcp](https://github.com/floci-io/floci-gcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
