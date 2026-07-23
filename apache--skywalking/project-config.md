---
trigger: always_on
description: This file provides guidance for AI assistants working with the Apache SkyWalking codebase.
---

# CLAUDE.md - AI Assistant Guide for Apache SkyWalking

This file provides guidance for AI assistants working with the Apache SkyWalking codebase.

## Project Overview

Apache SkyWalking is an open-source APM (Application Performance Monitoring) system designed for microservices, cloud-native, and container-based architectures. It provides distributed tracing, service mesh telemetry analysis, metrics aggregation, alerting, and observability capabilities.

## Repository Structure

```
skywalking/
├── oap-server/                    # OAP (Observability Analysis Platform) backend server
│   ├── server-core/               # Core module with fundamental services
│   ├── server-library/            # Shared libraries (module system, util, etc.)
│   ├── server-receiver-plugin/    # Data receivers (gRPC, HTTP, Kafka, etc.)
│   ├── server-storage-plugin/     # Storage implementations (BanyanDB, Elasticsearch, etc.)
│   ├── server-cluster-plugin/     # Cluster coordination (Zookeeper, K8s, etc.)
│   ├── server-query-plugin/       # Query interfaces (GraphQL)
│   ├── server-alarm-plugin/       # Alerting system
│   ├── server-fetcher-plugin/     # Data fetchers
│   ├── server-configuration/      # Dynamic configuration providers
│   ├── oal-grammar/               # OAL (Observability Analysis Language) grammar
│   ├── oal-rt/                    # OAL runtime
│   ├── mqe-grammar/               # MQE (Metrics Query Engine) grammar
│   ├── mqe-rt/                    # MQE runtime
│   ├── server-testing/             # Shared test utilities (DSL test framework)
│   ├── analyzer/                  # Log and trace analyzers
│   ├── ai-pipeline/               # AI/ML pipeline components
│   ├── exporter/                  # Data export functionality
│   └── server-tools/              # Standalone tools (profile exporter) with mock providers
├── apm-protocol/                  # Protocol definitions (submodule)
│   └── apm-network/               # gRPC/Protobuf network protocols
├── skywalking-ui/                 # Web UI (submodule - skywalking-booster-ui)
├── apm-webapp/                    # Web application packaging
├── apm-dist/                      # Distribution packaging
├── docs/                          # Documentation
├── docker/                        # Docker build files
├── test/                          # E2E and integration tests
└── tools/                         # Development tools
```

## Architecture & Key Concepts

### Module System
SkyWalking uses a custom module/provider architecture based on Java SPI:

- **ModuleDefine**: Declares a module and its required services
- **ModuleProvider**: Implements a module with specific technology/approach
- **Service**: Interface that modules expose to other modules

Key pattern:
```java
public class XxxModule extends ModuleDefine {
    public Class[] services() {
        return new Class[] { XxxService.class };
    }
}

public class XxxModuleProvider extends ModuleProvider {
    public void prepare() { /* initialize */ }
    public void start() { /* start services */ }
}
```

### Core Concepts
- **OAL (Observability Analysis Language)**: DSL for defining metrics aggregation rules
- **MQE (Metrics Query Engine)**: DSL for querying metrics
- **LAL (Log Analysis Language)**: DSL for log processing
- **MAL (Meter Analysis Language)**: DSL for meter data processing
- **Source/Scope**: Data model definitions for telemetry data
- **Stream Processing**: Metrics, Records, and TopN processing pipelines

### Data Flow
1. Agents/Collectors send data via gRPC/HTTP/Kafka
2. Receiver plugins parse and validate data
3. Analysis engine processes data using OAL/LAL/MAL
4. Storage plugins persist aggregated data
5. Query plugins serve data to UI/API

## Code Style & Conventions

### Checkstyle Rules (enforced via `apm-checkstyle/checkStyle.xml`)

**Prohibited patterns:**
- No `System.out.println` - use proper logging (SLF4J)
- No `@author` tags - ASF projects don't use author annotations
- No Chinese characters in source files
- No tab characters (use 4 spaces)
- No star imports (`import xxx.*`)
- No unused or redundant imports
- No empty statements (standalone `;`)
- No fully-qualified class names inline in code — always add an `import` statement and
  use the short name. Acceptable exceptions: (a) two classes with the same simple name
  would collide if both imported, (b) the class appears exactly once in a Javadoc
  `{@link}` where the short name would be ambiguous to the reader. Field declarations,
  method signatures, local variables, and generic type arguments should always use the
  imported short name — `private RemoteClientManager rcm;`, not `private
  org.apache.skywalking.oap.server.core.remote.client.RemoteClientManager rcm;`.
- No one-line delegate methods. A wrapper whose only body is a single forwarding call
  to another class (`return Other.foo(a, b);`) adds a hop without value. Inline the
  call at the use site, or call the underlying object directly (including via method
  reference: `obj::foo` instead of `this::wrapperOfFoo`).

**Required patterns:**
- `@Override` annotation required for overridden methods
- `equals()` and `hashCode()` must be overridden together
- Javadoc `@param`, `@return`, `@throws` must have descriptions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [apache/skywalking](https://github.com/apache/skywalking) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
