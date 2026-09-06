---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Mindconnect (repo `mindconnect-ai/mindconnect`) is a multi-module Maven monorepo for building LLM-powered agents.
It is built with Java 21; the deployable apps use Spring Boot 3.5.x, the core
libraries are plain Java. The repo is organized into four areas plus one shared
parent POM.

**Each area is fully self-contained and works independently** — there is no required
adoption of the whole stack. `workflow` is a small, fast workflow engine that embeds as
a plain library into any Java application. `taskqueue` is a dependency-free task queue.
`agents` is a standalone agent platform. Only `common/` is a shared dependency, and its
modules can be pulled in individually.

The server-driven UI framework (semantic-ui) lives in its own repository —
[mindconnect-ai/mc-semantic-ui](https://github.com/mindconnect-ai/mc-semantic-ui) —
and is consumed here as published Maven artifacts (`semantic-ui.version` in
`mc-java-parent/pom.xml` pins the version).

Each area has its own README — read it first when working in that area:

- [agents/README.md](agents/README.md)
- [workflow/README.md](workflow/README.md)
- [taskqueue/README.md](taskqueue/README.md)
- [common/README.md](common/README.md)

## Repository Layout

The root `pom.xml` is an aggregator that builds, in order: the parent POMs, the shared
`common/` libraries, then `taskqueue/`, `workflow/` and `agents/`.

- **`mc-java-parent/`** — the one repo-wide parent POM (install first; base
  Java 21 config, plugin management, Lombok, Spring Boot BOM, the Maven Central
  `release` profile). Each area's own parent lives inside that area
  (`agents/mc-agents-parent`, `workflow/mc-workflow-parent`) so an area can
  move to its own repo wholesale.

- **`common/`** — shared, dependency-light libraries
  - `mc-common`: domain primitives and shared types
  - `mc-schema`: typed model of the JSON-Schema subset used for tool signatures and workflow params
  - `mc-file-manager`: file storage / upload / download utilities
  - `mc-webscraper`: web scraping and content extraction (jsoup + Playwright backends)
  - `mc-pathaccessor`: navigate / read / write nested object & JSON paths
  - `mc-script-mini`: minimal embeddable script runner (MiniScript)
  - `mc-initial-data`: seed an app from bundled classpath resources on first run

- **`taskqueue/`** — dependency-free task queue (virtual-thread workers,
  suspend/resume, retries, cron scheduling); the agents sub-agent engine runs on it
  - `mc-task-queue`: core — ports + local implementation, plus the channel (observation) package
  - `mc-task-queue-schedule`: cron addon
  - `mc-task-queue-jdbc`: Postgres-backed stores (SKIP LOCKED claims, leases)
  - `mc-taskqueue-demo-app`, `mc-taskqueue-cluster-demo`: runnable demos

- **`workflow/`** — small, fast workflow engine, embeddable as a library into any Java app
  (steps, control flow, scoped variables, MiniScript/SpEL)
  - `mc-workflow`: core engine
  - `mc-workflow-code-{beanshell,javascript,groovy,jython}`: per-language script steps
  - `mc-workflow-jackson`: JSON (de)serialization of definitions
  - `mc-workflow-persistence`: pause / resume workflow instances
  - `mc-workflow-step-form`: a halt step that carries a semantic-ui form
  - `mc-workflow-dsl-puml`: build workflows from PlantUML
  - `mc-workflow-admin-rest`, `mc-workflow-admin-app`: embeddable workflow admin UI + standalone app
  - `mc-workflow-ui-diagram`, `mc-workflow-ui-diagram-app`, `mc-workflow-swing-ui`: UIs
  - `mc-workflow-spi-lookup`, `mc-workflow-test`: pluggable-step lookup, test fixtures

- **`agents/`** — the agentic runtime (the centerpiece), split along a
  core/adapter seam (`-core` modules carry domain + ports, siblings the implementations)
  - `mc-agents-parent/` — the area parent POM
  - `core/` — libraries (no runnable apps)
    - `mc-agent-runtime-core` / `mc-agent-runtime`: execution engine (turn loop, tool
      dispatch, sub-agent calls, approvals) / its adapters (file & in-memory repos,
      Pebble prompt renderer, tokenizer)
    - `mc-agent-memory-strategies`: working-memory strategies (window, summarizing window, auto-compact)
    - `mc-llm-gateway-core` / `mc-llm-gateway`: LLM abstraction ports / provider adapters + routing
    - `mc-message-repository-core` / `mc-message-repository`: conversation & message storage
    - `mc-agent-protocol` (+ `-openai`, `-mc-runtime`): protocol vocabulary and backend adapters
    - `mc-agent-tool-spi`: what a tool is — `Tool`, `ToolFactory`, `MultiToolProvider`, registry
    - `mc-agent-tools*`: built-in tool providers (filesystem/workspace/bash, code, document, web, web-browser, workflow, gmail)
    - `mc-credentials`: credential storage for tools & providers
    - `mc-mcp-proxy`: proxy for Model Context Protocol servers
  - `vectorstore/` — the knowledge layer: `mc-vector-store` (SPI + memory backend),
    `mc-vector-store-pgvector`, `mc-vector-store-tools`, `mc-file-store-core` / `mc-file-store`
  - `adapter/` — alternative implementations of the core ports; `postgres/mc-*-pg`
    modules store domain objects as JSONB documents via `common/mc-jdbc`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mindconnect-ai/mindconnect](https://github.com/mindconnect-ai/mindconnect) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
