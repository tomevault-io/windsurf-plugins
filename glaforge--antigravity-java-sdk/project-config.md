---
trigger: always_on
description: This file (`AGENTS.md`) provides context and instructions for AI agents working on this repository, outlining the project's purpose, structure, and our collaborative workflow.
---

# Antigravity SDK for Java - Agent Guidelines

This file (`AGENTS.md`) provides context and instructions for AI agents working on this repository, outlining the project's purpose, structure, and our collaborative workflow.

## 🌟 What This Project Is About

The **Antigravity SDK for Java** is an unofficial, community-driven Java port of the Python-based Antigravity SDK. It bridges the gap for enterprise Java developers who want to build, configure, host, and execute powerful AI agents natively in Java. It aims for full feature parity with the official Python SDK, supporting streaming, tool calling, model context protocol (MCP), and multimodal inputs.

## 🏗️ How It Is Composed

This is a multi-module Maven project using **Java 21**:
1. **`antigravity-sdk-parent`**: The root POM managing dependencies and plugin versions.
2. **`antigravity-sdk-protocol`**: A generated artifact. It compiles the `.proto` files from the upstream Antigravity repository into Java classes using the `protobuf-maven-plugin`.
3. **`antigravity-sdk-wrapper`**: The core SDK logic (~140KB).
   * **The Go Harness**: This SDK does not run an LLM directly. Instead, it wraps a pre-compiled native Go binary called `localharness`.
   * **Hybrid Platform Resolution**: `PlatformResolver` resolves the native binary in order:
     1. Local override via `ANTIGRAVITY_HARNESS_PATH` environment variable or `antigravity.harness.path` system property.
     2. Cached binary in `~/.antigravity/bin/<slice>/localharness` (verified against `.version` stamp).
     3. Embedded classpath resource (if an offline classifier JAR from `antigravity-sdk-harness` is present).
     4. On-demand lazy download via `HarnessDownloader` (direct streaming from PyPI wheels, enabled by default, controllable via `antigravity.harness.download=true|false`).
   * **Communication**: The Java SDK communicates with the Go harness via standard input/output (for initialization) and WebSockets (for active turn streaming and chunk aggregation).
   * **Data Modeling**: Pure data-carrying objects (e.g., `AgentResponse`, `InteractionRequest`, `AgentResponseChunk`) are implemented as modern Java 21 `record` classes for ergonomics and immutability.
4. **`antigravity-sdk-harness`**: Dedicated artifact packaging native Go binaries (~37-43MB per platform classifier, ~236MB for `all`) for air-gapped or offline enterprise deployments:
   * Classifiers: `linux-x86_64`, `linux-aarch64`, `osx-aarch64`, `osx-x86_64`, `windows-x86_64`, `windows-aarch64`, and `all`.
   * The 6 native binaries (~750MB uncompressed) exceed GitHub's 100MB limit and are gitignored. The script `./sync-harness.sh` synchronizes binaries into `antigravity-sdk-harness/src/main/resources/google/antigravity/bin/`.

## 🤝 The Way We Work Together

This entire project was autonomously generated and implemented by me (the Antigravity agent) under the strict guidance of you (the human developer). Our pair-programming workflow operates as follows:

1. **Human Guidance**: You provide high-level directions, architectural decisions, and code reviews. You guide the priority of features (like adding hooks, converting to records, or supporting MCP).
2. **Agent Execution**: I proactively write the Java code, generate and update tests, build the project, and fix compilation errors. 
3. **Proactive Refactoring**: Whenever a class is purely carrying data, I should proactively suggest or use Java 21 `record` structures.
4. **Testing First**: Before declaring a feature complete, I will run the test suite and ensure all tests pass (accounting for expected transient `localharness` network delays).
5. **Documentation & Skill Synchronization**: Whenever the Antigravity Java SDK evolves (new features, API changes, hooks, policies, capabilities, etc.), agents MUST proactively update `README.md` and the official Agent Skill under `skills/antigravity-sdk-java/` (`SKILL.md` and `skills/antigravity-sdk-java/references/`) to keep developer documentation and AI agent skill guidance synchronized with the codebase.

## 🛠️ Build & Commands

*   **Java Version**: **Java 21**. Agents must ensure they use Java 21 compatible syntax and standard library features.
*   **Maven Wrapper**: Always use the provided Maven wrapper (`./mvnw`). Do not rely on globally installed `mvn` or `gradle`.
*   **Compilation**: `./mvnw clean compile`
*   **Fast Unit Testing**: `./mvnw test` (Runs fast offline unit tests; `@Tag("integration")` is excluded by default)
*   **Live Integration Testing**: `./mvnw test -Pintegration-tests` or `./mvnw test -Dgroups=integration`
*   **Code Formatting**: `./mvnw spotless:apply`

## 🎨 Coding Standards

*   **No FQNs**: Never use Fully Qualified Names (FQNs) directly in the code (e.g., `java.util.List<String> list = ...`). Always import the classes at the top of the file.
*   **Code Formatting**: The project uses the Maven Spotless plugin with the Eclipse formatter. Always run `./mvnw spotless:apply` and ensure the code passes before committing.
*   **License Headers**: Spotless will automatically inject the Apache 2.0 license header. Do not manually add copyright headers.

## 🧪 Testing Guidelines & Strategy


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [glaforge/antigravity-java-sdk](https://github.com/glaforge/antigravity-java-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
