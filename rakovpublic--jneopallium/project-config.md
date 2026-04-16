---
trigger: always_on
description: **jneopallium** is a Java-based neural network simulation framework that models realistic neuron behavior. It supports local, HTTP-cluster, and gRPC-based distributed execution. The project is published research (doi: SR24703042047) and is under active development.
---

# CLAUDE.md — AI Assistant Guide for jneopallium

## Project Overview

**jneopallium** is a Java-based neural network simulation framework that models realistic neuron behavior. It supports local, HTTP-cluster, and gRPC-based distributed execution. The project is published research (doi: SR24703042047) and is under active development.

- **License**: BSD 3-Clause (Copyright 2018 Dmytro Rakovskyi)
- **Language**: Java 11+
- **Build System**: Maven (multi-module)
- **Package Root**: `com.rakovpublic.jneuropallium`

---

## Repository Structure

```
jneopallium/
├── pom.xml                   # Parent POM — declares modules
├── master/                   # Spring Boot WAR — orchestration/API layer
│   ├── pom.xml
│   └── src/main/
│       ├── java/             # Controllers, services, config beans
│       └── resources/
│           ├── application.yml
│           └── log4j2.yml
├── worker/                   # Core neuron engine — JAR with dependencies
│   ├── pom.xml
│   └── src/main/
│       ├── java/             # Neuron model, layers, signals, study algorithms
│       ├── proto/            # gRPC service definitions
│       └── resources/
│           ├── config.properties   # Primary runtime configuration
│           └── log4j2.yml
├── doc/                      # Logo, research PDF
├── .github/
│   ├── workflows/codeql.yml  # CodeQL static analysis CI
│   └── ISSUE_TEMPLATE/       # Bug report, feature request templates
├── TestPlanPhase1.md         # Test plan and completion status
├── TestPlanPhase2.md
├── WorkDiary.md              # Development diary/progress log
└── README.md                 # Project overview and architecture
```

---

## Module Overview

### `master` module (Spring Boot WAR)
The orchestration layer — exposes REST APIs and manages worker nodes.

| Component | Package | Purpose |
|-----------|---------|---------|
| Controllers | `...controller` | REST endpoints: nodes, layers, inputs, config, signals, class loading |
| Services | `...service` | Business logic: config, node management, signals, storage |
| Config | `...config` | Spring beans, property holders |
| Entry Point | `Application.java` | `@SpringBootApplication` bootstrap |

**Packaged as**: `jneuronnetmaster.war`

### `worker` module (Executable JAR)
The core neuron execution engine — runs neuron networks directly.

| Package | Purpose |
|---------|---------|
| `application/` | Entry points: `LocalApplication`, `HttpClusterApplication`, `GRPCServerApplication`, `Entry` |
| `net/neuron/` | Core interfaces: `INeuron`, `IDendrites`, `IAxon`, `ISignalProcessor`, `IWeight`, `INeuronRunner` |
| `net/layers/` | Layer management: `ILayer`, `ILayersMeta`, `IResultLayer` |
| `net/signals/` | Signal handling: `ISignal`, `IInputSignal`, `IResultSignal`, `ISignalStorage`, `IInputLoadingStrategy` |
| `net/study/` | Learning: `ILearningAlgo`, `IDirectLearningAlgorithm`, `IObjectLearningAlgo`, `IResultComparingStrategy` |
| `net/storages/` | Storage abstraction: `IStorage` and file/in-memory/Redis/JSON implementations |
| `util/` | Utilities: `NeuronNetStructureGenerator`, `JarClassLoaderService`, `NeuronParser`, `Context` |

**Entry class**: `Entry.java` — accepts 4 command-line arguments at startup.

---

## Technology Stack

| Technology | Version | Use |
|------------|---------|-----|
| Java | 11 | Primary language |
| Spring Boot | 2.5.12 / 2.7.18 | Master module web framework |
| gRPC | 1.75.0 | Distributed worker communication |
| Protocol Buffers | 3.25.5 | gRPC message definitions |
| Kafka | 3.9.1 | Async input signal source |
| Redis (Jedis) | 5.0.2 | Distributed state storage |
| Jackson | 2.12.7.1 | JSON serialization |
| Gson | 2.8.9 | Alternate JSON handling |
| Log4j 2 | 2.25.3 | Logging |
| JUnit | 4.13.1 | Testing |
| Maven | — | Build system |

---

## Build Commands

```bash
# Build entire project (all modules)
mvn clean package

# Build without running tests
mvn clean package -DskipTests

# Build specific module
mvn clean package -pl worker
mvn clean package -pl master

# Run tests
mvn test

# Run specific test class
mvn test -Dtest=ClassName

# Install to local Maven cache
mvn clean install
```

**Build outputs:**
- `master/target/jneuronnetmaster.war`
- `worker/target/worker-<version>-jar-with-dependencies.jar`

---

## Running the Application

### Local (single machine)
```bash
java -jar worker/target/worker-*-jar-with-dependencies.jar <arg1> <arg2> <arg3> <arg4>
```
Uses `LocalApplication` — runs the full neuron net in-process.

### HTTP Cluster
Uses `HttpClusterApplication` — distributes work across HTTP-accessible worker nodes managed by the master Spring Boot app.

### gRPC Cluster
Uses `GRPCServerApplication` — exposes a gRPC server defined in `worker/src/main/proto/jneoapalliumservice.proto`.

**Master module** (manages distributed workers):
```bash
java -jar master/target/jneuronnetmaster.war
```

---

## Configuration

### Worker: `worker/src/main/resources/config.properties`
Primary runtime configuration. Key properties:

```properties
# Execution control
configuration.maxRun=2

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rakovpublic) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
