---
trigger: always_on
description: Repo: https://github.com/platonai/Browser4
---

# Repository Guidelines

Repo: https://github.com/platonai/Browser4

<!-- TOC -->
**Table of Contents**
- [Project Overview](#project-overview)
- [Quick Start](#quick-start)
- [Project Structure](#project-structure)
- [Key APIs and Concepts](#key-apis-and-concepts)
- [Code Style Guidelines](#code-style-guidelines)
- [Testing Guidelines](#testing-guidelines)
- [Configuration](#configuration)
- [Development Principles](#development-principles)
- [Definition of Done](#definition-of-done-pr-checklist)
- [Common Issues & Troubleshooting](#common-issues--troubleshooting)
- [Documentation References](#documentation-references)
- [Claude-Specific Guidance](#claude-specific-guidance)
<!-- /TOC -->

---

## Project Overview

**Browser4** is a lightning-fast, coroutine-safe browser engine for AI agents. It provides:

- **Browser Agents** — Fully autonomous browser agents that reason, plan, and execute end-to-end tasks
- **Browser Automation** — High-performance automation for workflows, navigation, and data extraction
- **Machine Learning Agent** — Learns field structures across complex pages without consuming tokens
- **Extreme Performance** — Fully coroutine-safe; supports 100k ~ 200k complex page visits per machine per day
- **Data Extraction** — Hybrid of LLM, ML, and selectors for clean data across chaotic pages

## Quick Start

### Prerequisites
- Java 17+
- Latest Google Chrome

### Build Commands

**Linux/macOS:**
```bash
chmod +x mvnw
./mvnw -q -DskipTests
```

**Windows (PowerShell):**
```powershell
.\mvnw.cmd -q -D"skipTests"
```

**Windows (cmd):**
```cmd
mvnw.cmd -q -DskipTests
```

### Run Tests

**Core module tests (Linux/macOS):**
```bash
./mvnw -pl browser4-core -am test -Dsurefire.failIfNoSpecifiedTests=false
```

**Core module tests (Windows PowerShell):**
```powershell
.\mvnw.cmd -pl browser4-core -am test -D"surefire.failIfNoSpecifiedTests=false"
```

### Recommended Build Scripts
- Windows: `bin/build.ps1 [-test]`
- Linux/macOS: `bin/build.sh [-test]`

> **Note for Linux/macOS:** Many scripts in this repo are PowerShell (`.ps1`) files (e.g., `bin/test.ps1`, `bin/build.ps1`). To run them on Linux/macOS, install PowerShell if not already installed, then use `pwsh`:
> ```bash
> # Install PowerShell (Ubuntu/Debian)
> sudo apt-get install -y powershell
> # Or via snap
> sudo snap install powershell --classic
>
> # Run a PowerShell script
> pwsh bin/test.ps1 fast
> pwsh bin/build.ps1 -test
> ```

## Project Structure

| Module                                 | Description |
|----------------------------------------|-------------|
| `browser4-core`                        | Core engine: sessions, scheduling, DOM, browser control |
| `browser4-dependencies`                | BOM and dependency alignment |
| `browser4-tools`                       | Operational tools and launch helpers |
| `browser4-agentic`                     | AI agents implementation, MCP, skills registration |
| `browser4-agent-tools`                 | High-level agent tools: scraping, crawling, stateful page interaction |
| `browser4-rest`                        | Spring Boot REST layer & command endpoints |
| `cli/*`                                | Browser4 CLI + skill assets (`cli/browser4-cli`, `cli/skill`) |
| `browser4-apps/*`                      | Product packaging and the unified launcher (`browser4-apps/browser4-standalone`, `target/Browser4.jar`) |
| `examples/*`                           | Runnable examples (`examples/browser4-examples`) |
| `browser4-tests`                       | E2E & heavy integration & scenario tests |
| `browser4-tests/browser4-tests-common` | Shared test base classes and utilities |
| `cdp-protocol`                         | Chrome DevTools Protocol JSON definitions (`browser_protocol.json`, `js_protocol.json`) |
| `coworker/`                            | File-queue automation system for task-driven AI workflows |

## Key APIs and Concepts

### Sessions
```kotlin
// Create a session
val session = AgenticContexts.createSession()

// Create an agent
val agent = AgenticContexts.getOrCreateAgent()
```

### Core API Classes
- `WebDriver` — Browser control interface with human-like behaviors
- `PulsarSession` → `AgenticSession` — Page loading, parsing, and extraction
- `LoadOptions` — CLI-style URL parameters for page loading
- `BrowserPerceptiveAgent` — AI agent implementation

### Load Options
URL parameters control page loading behavior:
```kotlin
val page = session.load(url, "-expires 1d -refresh -parse")
```

Key options:
- `-expires <duration>` — Page expiration time
- `-refresh` — Force page refresh
- `-parse` — Activate parsing subsystem
- `-outLink <selector>` — Extract links matching selector

## Code Style Guidelines

### Kotlin Conventions
- Prefer immutable `data class`
- Use explicit return types
- Apply null-safety patterns (`require`/`check`/`?:`)
- Public APIs require KDoc documentation
- Store AI generated task docs in `docs-dev/copilot/`

**KDoc Template:**
```kotlin
/**
 * Brief description of what the function does.
 *
 * @param paramName Description of the parameter.
 * @return Description of the return value.
 * @throws ExceptionType When this exception is thrown.
 */
fun functionName(paramName: Type): ReturnType {
    require(paramName.isValid) { "paramName must be valid" }

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [platonai/Browser4](https://github.com/platonai/Browser4) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
