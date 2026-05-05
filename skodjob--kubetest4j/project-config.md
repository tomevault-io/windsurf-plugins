---
trigger: always_on
description: **kubetest4j** is a Java testing library for Kubernetes and OpenShift clusters. It provides declarative, annotation-based testing with automatic resource management, multi-context cluster support, and integrated log/metrics collection.
---

# AGENTS.md - AI Coding Agent Guide for kubetest4j

## Project Overview

**kubetest4j** is a Java testing library for Kubernetes and OpenShift clusters. It provides declarative, annotation-based testing with automatic resource management, multi-context cluster support, and integrated log/metrics collection.

- **Organization:** skodjob
- **License:** Apache 2.0
- **Java:** 21+ (CI tests on 21 and 25)
- **Repository:** https://github.com/skodjob/kubetest4j

## Build & Test Commands

```bash
./mvnw install                        # Build all modules
./mvnw install -pl kubetest4j         # Build single module
./mvnw test -pl <module>              # Unit tests for a module
./mvnw checkstyle:check -pl <module>  # Verify checkstyle compliance
./mvnw spotbugs:spotbugs              # Static analysis
./mvnw verify -P integration          # Integration tests (requires cluster)
```

**Before committing, always run `./mvnw checkstyle:check` on changed modules.** Checkstyle is enforced as an error in CI.

## CI Pipeline & PR Checks

PRs are reviewed by **CodeRabbit** (AI reviewer) and quality-gated by **SonarCloud**:
- **SonarCloud:** New code must have **>80% test coverage** (enforced). JaCoCo coverage is collected from `./mvnw verify -P integration` — both unit tests (`*Test.java`) and integration tests (`*IT.java`) count toward coverage.
- **CI builds:** `./mvnw install` + `./mvnw spotbugs:spotbugs` on Java 21 and 25.
- **CodeQL:** GitHub default setup for security vulnerability scanning (enabled in repo settings, no workflow file).
- **Scorecard:** OpenSSF supply-chain security analysis (`scorecard.yml`).
- **Verification:** SonarCloud scan runs on a separate workflow (`verify.yaml`).

When writing new code, add unit tests to cover it. Integration tests running against a real Kind cluster provide additional coverage.

## Module Structure

```
kubetest4j/                  # Core library - clients, resource management, utilities
kubernetes-resources/        # ResourceType implementations for K8s native resources
openshift-resources/         # ResourceType implementations for OpenShift/OLM resources
junit-extension/             # JUnit 5 extension with @KubernetesTest annotation
log-collector/               # Pod log/description/YAML collection utility
metrics-collector/           # Prometheus metrics scraping from pods
test-examples/               # Integration test examples (reference for patterns)
```

**Module dependencies:** everything depends on `kubetest4j` (core). `test-examples` depends on all modules (test scope only).

## Architecture & Key Abstractions

### ResourceType<T> (`kubetest4j/.../interfaces/ResourceType.java`)
Core interface for Kubernetes resource lifecycle. Implement `getKind()`, `create()`, `update()`, `delete()`, `replace()`, `isReady()`, `isDeleted()`, `getTimeoutForResourceReadiness()`.

**To add a new resource type:** follow the pattern in `kubernetes-resources/.../resources/DeploymentType.java`.

### KubeResourceManager (`kubetest4j/.../resources/KubeResourceManager.java`)
Singleton managing resource lifecycle and LIFO-stack cleanup:
- `KubeResourceManager.get()` / `getForContext("name")` — default or per-context instance
- `createResourceWithWait(T)` / `createResourceWithoutWait(T)` / `createResourceAsyncWait(T)`
- `replaceResourceWithRetries(T, Consumer<T>)` — auto-retries on 409 conflicts
- `deleteResources()` / `deleteResourceWithWait(T)` — cleanup methods
- Tracks resources per `contextId + testDisplayName` in `STORED_RESOURCES`
- Uses `Semaphore` (default 50) for async operation throttling
- Uses virtual threads (`Executors.newVirtualThreadPerTaskExecutor()`)
- Global callbacks: `addCreateCallback()` / `addDeleteCallback()` — fire on resource create/delete

**Thread safety:** `ConcurrentHashMap` for instances/caches, `CopyOnWriteArrayList` for callbacks, `ThreadLocal` for cluster context, `Stack` for per-test resource tracking.

### KubeClient (`kubetest4j/.../clients/KubeClient.java`)
Wrapper around Fabric8 `KubernetesClient`:
- `new KubeClient()` — auto-configure from env vars/kubeconfig
- `new KubeClient(kubeconfigPath)` — from kubeconfig file
- `KubeClient.fromUrlAndToken(url, token)` — from API URL + bearer token (generates temp kubeconfig cleaned up via JVM shutdown hook)

### KubeCmdClient / Kubectl / Oc (`kubetest4j/.../clients/cmdClient/`)
Abstraction for kubectl/oc CLI operations. `BaseCmdKubeClient` is the base; `Kubectl` and `Oc` provide implementation-specific behavior (e.g., `oc new-project` vs `kubectl create namespace`).

### JUnit Extension (`junit-extension/.../KubernetesTestExtension.java`)

**`@KubernetesTest` annotation** (see `junit-extension/.../annotations/KubernetesTest.java` for full attribute list):
- `resourceTypes` — declarative ResourceType registration
- `cleanup` — `CleanupStrategy.AUTOMATIC` or `MANUAL`
- `collectLogs` / `logCollectionStrategy` / `logCollectionPath` / `collectPreviousLogs` — log collection config
- `collectNamespacedResources` / `collectClusterWideResources` — what to collect
- `storeYaml` / `yamlStorePath` — persist created resource YAMLs

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [skodjob/kubetest4j](https://github.com/skodjob/kubetest4j) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
