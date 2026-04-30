---
trigger: always_on
description: Ganglia is a **Java 25** AI Agent framework built on **Vert.x 5.0.6**, designed for high-performance, non-blocking agentic workflows. It follows a **Hexagonal (Ports & Adapters)** architecture with a robust ReAct reasoning loop, pluggable memory via SPI, plan mode with disk persistence, and multi-frontend support (Terminal, WebUI, IM channels).
---

# AGENTS.md — Ganglia Project Instructions

## 1. Project Overview

Ganglia is a **Java 25** AI Agent framework built on **Vert.x 5.0.6**, designed for high-performance, non-blocking agentic workflows. It follows a **Hexagonal (Ports & Adapters)** architecture with a robust ReAct reasoning loop, pluggable memory via SPI, plan mode with disk persistence, and multi-frontend support (Terminal, WebUI, IM channels).

**Version:** 0.1.7-SNAPSHOT

## 2. Technology Stack

|    Category     |                                   Technology                                    |
|-----------------|---------------------------------------------------------------------------------|
| Runtime         | Java 25-zulu (SDKMAN! managed)                                                  |
| Core            | Vert.x 5.0.6 (Reactive, Non-blocking I/O)                                       |
| LLM Integration | Native OpenAI & Anthropic protocol, Codex (OkHttp SSE)                          |
| Terminal UI     | JLine 3.25.1, CommonMark (ANSI Markdown)                                        |
| Web UI          | React 18 + TypeScript (Vite), Vert.x Web (WebSocket + JSON-RPC 2.0)             |
| Networking      | Vert.x WebClient 5.0.6                                                          |
| Logging         | SLF4J 2.0.16 + Log4j2                                                           |
| Testing         | JUnit 5, Mockito, Vertx-JUnit5, E2E Simulation Harness                          |
| Code Quality    | Spotless (Google Java Format), Checkstyle (google_checks.xml), SpotBugs, JaCoCo |

## 3. Module Structure

```
ganglia-parent (pom.xml)
├── ganglia-harness                  # Core: kernel, ports, infrastructure (no memory impl)
├── ganglia-local-file-memory        # File-based memory SPI implementation
├── ganglia-sqlite-memory            # SQLite-backed memory SPI implementation
├── ganglia-observability            # Independent Trace Studio backend and REST API
├── ganglia-trajectory               # Unified observation event stream and JSONL trace management
├── coding-agent/                    # Directory grouping (NOT a Maven module)
│   ├── ganglia-coding               # Coding agent builder + tools (bash, file-edit, web-fetch)
│   ├── ganglia-coding-web           # WebSocket + JSON-RPC 2.0 web UI backend
│   ├── ganglia-terminal             # JLine 3 terminal UI
│   ├── ganglia-swe-bench            # SWE-bench evaluation with Docker sandboxing
│   └── ganglia-coding-webui         # React multi-page frontend (NOT a Maven module)
├── trading-agent/                   # Directory grouping
│   ├── ganglia-trading              # Multi-agent adversarial trading system
│   ├── ganglia-trading-web          # WebSocket backend for trading dashboard
│   └── ganglia-trading-webui        # React frontend for trading UI (NOT a Maven module)
├── claw-agent/                      # Directory grouping
│   ├── ganglia-claw                 # IM-integrated agent orchestration (channels, ACP, scheduling)
│   ├── ganglia-claw-web             # Management dashboard REST API
│   └── ganglia-claw-app             # Application layer: daemon lifecycle, multi-agent verticles
├── e2e-test                         # E2E tests with real LLM calls (trading/, coding/, channel/, gateway/)
├── integration-test                 # Component integration tests with mock models
└── ganglia-example                  # Demo apps (WebUIDemo)
```

> **Note:** `coding-agent/`, `trading-agent/`, and `claw-agent/` are plain directories, not Maven aggregators. Sub-modules are listed directly in the root `pom.xml`. `*-webui` projects are standalone Vite/React projects, not managed by Maven.

### Dependency Graph

```
ganglia-harness
    ↑
ganglia-local-file-memory, ganglia-sqlite-memory, ganglia-trajectory
    ↑
ganglia-coding, ganglia-trading, ganglia-claw, ganglia-observability
    ↑
ganglia-coding-web, ganglia-trading-web, ganglia-claw-web
    ↑
ganglia-claw-app (combines claw + trading + coding + sqlite-memory)
    ↑
integration-test, ganglia-example, ganglia-swe-bench
```

## 4. Build & Development Commands

|             Command             |                                              Description                                              |
|---------------------------------|-------------------------------------------------------------------------------------------------------|
| `mvn clean install -DskipTests` | Full build (skip tests)                                                                               |
| `just test-backend`             | Unit tests: all Java modules (harness, memory, coding, trading, claw, etc.) |
| `just test-it`                  | Integration tests                                                                                     |
| `just test-it-one <ClassName>`  | Single integration test                                                                               |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stream-iori/ganglia](https://github.com/stream-iori/ganglia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
