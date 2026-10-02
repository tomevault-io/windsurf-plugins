---
trigger: always_on
description: This document serves as the canonical reference and behavioral guide for AI agents and developers contributing to `@gilfink/prompt-chain`.
---

# GEMINI.md — Project Tech Stack, Development Rules & Context References

This document serves as the canonical reference and behavioral guide for AI agents and developers contributing to `@gilfink/prompt-chain`.

---

## 1. Project Overview & Philosophy

- **Package Name**: `@gilfink/prompt-chain` (`On-Device Prompt API Chain AI Agents`)
- **Core Mission**: A production-grade, zero-dependency ES6 library that brings enterprise-level AI agent orchestration directly to on-device browsers (`window.LanguageModel`), executing safely inside background Web Workers without main-thread UI freezing.
- **Zero Third-Party Usage (`Zero-Dependency Rule`)**:
  - The runtime library (`src/`) **must have ZERO external runtime dependencies** (`"dependencies": {}` in `package.json`).
  - Everything is built from scratch using pure ES6 modules and native Web APIs (`IndexedDB`, `Web Workers`, `Fetch API`, Chrome Prompt API `window.LanguageModel`).
  - Do not introduce NPM packages like `langchain`, `zod`, `axios`, or `@opentelemetry/sdk`. All enterprise capabilities (e.g., LCEL runnables, recursive JSON Schema validation, vector cosine similarity, OpenTelemetry OTLP tracing, and state graphs) are implemented natively within `src/`.

---

## 2. Project Tech Stack & Architecture

- **Language & Modules**: Modern ES6 JavaScript (`type: "module"`). All local imports must include explicit `.js` extensions (e.g., `import { BaseMessage } from "./messages.js"`).
- **Inference Engine**: Chrome Built-in AI (`window.LanguageModel`, `session.promptStreaming()`, `session.measureContextUsage()`). Supports hybrid local/cloud routing via `RunnableFallback`.
- **Build & Dev Tooling**:
  - `vite` (`^5.2.0`) for local development web server (`npm run dev`) and UMD/ESM library bundling (`npm run build`).
- **Asynchronous Web Worker Architecture**:
  - **`PromptChainHost` ([src/core/prompt-chain-host.js](file:///c:/Lectures/Demo/src/core/prompt-chain-host.js))**: Main UI thread session controller. Manages model readiness, event listeners (`CallbackManager`), and cross-thread message passing.
  - **`PromptChainWorker` ([src/core/prompt-chain-worker.js](file:///c:/Lectures/Demo/src/core/prompt-chain-worker.js))**: Background Web Worker runtime host. Orchestrates the 7-turn `ReActAgentExecutor` reasoning loop, executes multi-parameter structured tools, validates schemas, and runs custom LangChain Expression Language (LCEL) chains.
- **Persistent Local Storage & Universal Opener (`IndexedDB`)**:
  - **`openIndexedDB(dbName, requiredStores)` ([src/utils.js](file:///c:/Lectures/Demo/src/utils.js#L117-L170))**: The **Universal Database Migration Helper**. Always use this function to open IndexedDB databases. It opens the database without specifying a hardcoded version number (connecting cleanly to any existing version `v1`, `v2`, `v3`, etc. without `VersionError`), and dynamically upgrades (`db.version + 1`) only when new object stores are missing (`conversations`, `checkpoints`, `traces`, or `vectors`).
  - **`AgentMemory` ([src/core/agent-memory.js](file:///c:/Lectures/Demo/src/core/agent-memory.js))**: Stores object-oriented conversation histories (`HumanMessage`, `AIMessage`) and Human-in-the-Loop (HITL) checkpoints.
  - **`IndexedDBVectorStore` ([src/retrievers/indexeddb-vector-store.js](file:///c:/Lectures/Demo/src/retrievers/indexeddb-vector-store.js))**: Client-side vector database (`cosineSimilarity`) supporting semantic pruning for `SkillRetriever` and `ToolRetriever`.
  - **`IndexedDBTraceExporter` ([src/observability/exporters.js](file:///c:/Lectures/Demo/src/observability/exporters.js))**: Persists completed OpenTelemetry trace hierarchies locally (`AgentMemoryDB`, `traces` store).

---

## 3. Core Capabilities & Module Directory (`Context & Memory References`)

### LangChain Expression Language (LCEL) Runnables ([src/runnables/](file:///c:/Lectures/Demo/src/runnables))
- **Core Primitives**: [runnable.js](file:///c:/Lectures/Demo/src/runnables/runnable.js) (`.pipe()`, `.bind()`), [runnable-sequence.js](file:///c:/Lectures/Demo/src/runnables/runnable-sequence.js) (`RunnableSequence.from`), [runnable-parallel.js](file:///c:/Lectures/Demo/src/runnables/runnable-parallel.js) (`RunnableParallel`), [runnable-lambda.js](file:///c:/Lectures/Demo/src/runnables/runnable-lambda.js) (`RunnableLambda`), [runnable-passthrough.js](file:///c:/Lectures/Demo/src/runnables/runnable-passthrough.js) (`RunnablePassthrough.assign`).
- **`RunnableTokenBuffer` ([runnable-token-buffer.js](file:///c:/Lectures/Demo/src/runnables/runnable-token-buffer.js))**: Watermark context window monitoring (default 85%). Automatically truncates lengthy tool observations (`pruneObservation`) and summarizes rolling turn history (`session.measureContextUsage()`).
- **`RunnableInterrupt` ([runnable-interrupt.js](file:///c:/Lectures/Demo/src/runnables/runnable-interrupt.js))**: Human-in-the-Loop (HITL) safety rail. Suspends execution before running tools flagged with `{ requiresApproval: true }`, serializes exact state to IndexedDB (`checkpoints`), and emits `userApprovalRequired` to the UI.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gilf/prompt-chain](https://github.com/gilf/prompt-chain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
