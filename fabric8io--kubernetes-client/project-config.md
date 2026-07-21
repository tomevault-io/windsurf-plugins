---
trigger: always_on
description: Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.
---

# Fabric8 Kubernetes Client - AI Agents Instructions

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

This file provides guidance to AI coding agents (GitHub Copilot, Claude Code, etc.) when working with code in this repository.

## Project Overview

Fabric8 Kubernetes Client is a Java client library that provides access to the full Kubernetes and OpenShift REST APIs via a fluent DSL. The project is a large multi-module Maven build targeting Java 11+, with pluggable HTTP client implementations and extensive extension modules for Kubernetes ecosystem projects (Knative, Tekton, Istio, etc.).

## Working Effectively

### Bootstrap and Setup

```bash
# Clone the repository
git clone https://github.com/fabric8io/kubernetes-client.git
cd kubernetes-client

# Quick build without tests (recommended for setup)
make quickly
```

### Build Commands

The project uses a `Makefile` for common operations. Prefer make targets over raw Maven commands.

```bash
# Quick build without tests (default target, ~3-5 minutes)
make quickly

# Full build with tests (10-15 minutes)
# WARNING: This is a long-running command - DO NOT CANCEL
make install

# Clean build artifacts
make clean

# Format code (license headers + Java formatting) - run before committing
make format

# Generate/regenerate model classes from OpenAPI schemas
make generate-model

# Regenerate javadoc element-list files for cross-module linking
# Run after adding/removing packages in core modules
make generate-javadoc-links
```

**Direct Maven commands** (when make targets don't cover the use case):
```bash
# Build a specific module
mvn clean install -pl kubernetes-client-api -DskipTests -T 1C

# Build with dependencies
mvn clean install -pl kubernetes-client -am -DskipTests -T 1C

# Check formatting without applying
mvn spotless:check
```

### Testing

```bash
# Run all unit tests (5-10 minutes)
# WARNING: This is a long-running command - DO NOT CANCEL
mvn clean install

# Run tests for a specific module
mvn test -pl kubernetes-tests

# Run a single test class
mvn test -pl kubernetes-tests -Dtest=PodCrudTest

# Run integration tests (requires running Kubernetes cluster)
# WARNING: This is a long-running command - DO NOT CANCEL
mvn -Pitests -pl kubernetes-itests verify

# Run OpenShift-specific integration tests
mvn -Pitests -pl kubernetes-itests verify -Dtest="io.fabric8.openshift.**"
```

### Reproducing CI flakes locally

CI runs on 2-core GitHub Actions VMs with limited RAM. Surefire `forkCount=1C` and Vert.x event-loop sizing both scale with `Runtime.availableProcessors()`, so a flake that surfaces under CI scheduling pressure often won't reproduce with a plain `mvn test` on a 16+ core developer host. Run the test inside a constrained `docker` (or `podman`) container with the project and `~/.m2` bind-mounted — this is the closest local approximation to CI shape and almost always the fastest way to confirm or rule out a race.

```bash
# Constrained container — 2 cores, ~7 GB RAM, cgroup-enforced
docker run --rm \
  --cpus=2 --memory=7g --cpuset-cpus="0,1" \
  --user $(id -u):$(id -g) -e HOME=/tmp \
  -v "$PWD:/work" \
  -v "$HOME/.m2:/host-m2" \
  -w /work \
  maven:3.9-eclipse-temurin-11 \
  mvn -pl kubernetes-tests \
      -Dtest='SomeFlakyTest#someFlakyMethod' \
      -Dmaven.repo.local=/host-m2/repository \
      test
```

For harder contention, run a sibling `stress-ng` container pinned to the same `cpuset` as a noisy neighbor, then loop the test 25–50 times under that stress.

```bash
docker run --rm -d --name stressor --cpus=2 --cpuset-cpus="0,1" \
  alpine:latest sh -c 'apk add --no-cache stress-ng && stress-ng --cpu 4 --cpu-load 80 --timeout 600s'
```

Caveat: not every CI flake reproduces locally — GH Actions VMs add IO/network jitter that cgroup quota cannot simulate. After ~20 unsuccessful iterations, switch to instrument-and-wait (add timing logs on a debug branch and watch the next CI hit) rather than burning more local hours.

### Running Examples

```bash
# Examples are in kubernetes-examples module
cd kubernetes-examples
mvn exec:java -Dexec.mainClass="io.fabric8.kubernetes.examples.ListPods"
```

## Architecture

### Technical Structure

```
kubernetes-client/                    # Root project
├── kubernetes-client-api/            # Core API interfaces and abstractions
├── kubernetes-client/                # Main client implementation
├── kubernetes-model-generator/       # Generates model classes from K8s OpenAPI
├── java-generator/                   # Generates Java from CRDs
├── crd-generator/                    # CRD generator from Java classes
├── generator-annotations/            # Annotations for code generation
│
├── httpclient-jdk/                   # Java 11+ HTTP client implementation
├── httpclient-okhttp/                # OkHttp3 implementation
├── httpclient-jetty/                 # Jetty HTTP client
├── httpclient-vertx/                 # Vert.x HTTP client (default)
│
├── kubernetes-tests/                 # Unit tests with mock server

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fabric8io/kubernetes-client](https://github.com/fabric8io/kubernetes-client) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
