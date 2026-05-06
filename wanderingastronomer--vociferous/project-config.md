---
trigger: always_on
description: This file defines **authoritative, binding instructions** for GitHub Copilot, VS Code AI agents, and any autonomous or semi-autonomous coding assistants operating in this repository.
---

# Vociferous — Copilot & VS Code AI Instructions

## 0. Scope and Intent

This file defines **authoritative, binding instructions** for GitHub Copilot, VS Code AI agents, and any autonomous or semi-autonomous coding assistants operating in this repository.

These instructions exist to:

* Preserve long-term architectural integrity of the **Hybrid Python/JS Stack**
* Reduce cognitive and procedural friction for a **solo maintainer**
* Prevent AI-introduced process ceremony
* Encode project invariants and design intent explicitly

All guidance in this file is **normative**, not advisory. If a conflict exists between AI defaults and this document, **this document takes precedence**.

---

## 1. Philosophy

**Move carefully and Socratically.** Before writing code, understand context. Before changing a pattern, understand why it exists.

* Ask clarifying questions when requirements are ambiguous.
* Prefer small, verifiable increments over broad rewrites.
* When touching shared infrastructure (runtime, database, command bus, API), trace downstream consumers before editing.

---

## 2. Project Context (Always Read First)

**Vociferous** is a production-quality, cross-platform speech-to-text application.

**Supported Platforms**: Linux | macOS | Windows

Core stack:
* **Shell**: `pywebview` — GTK+WebKitGTK (Linux), Cocoa+WebKit (macOS), EdgeChromium (Windows)
* **Frontend**: Svelte 5 + Tailwind CSS v4 + Vite
* **Backend API**: Litestar (REST + WebSocket)
* **Core Logic**: Python 3.12+ (Command Bus, Application Coordinator)
* **ASR Runtime**: In-process background thread using `faster-whisper` (CTranslate2 Whisper backend)
* **SLM**: In-process background thread using `ctranslate2` Generator + `tokenizers`

The project is actively developed and maintained by a **single primary developer**. All workflow rules are calibrated accordingly.

**Note**: Docker containerization is Linux-only (requires X11/Wayland), but the application itself runs natively on all three platforms.

---

## 3. Architectural Invariants (Non-Negotiable)

### 3.1 Composition Root

* **`src/core/application_coordinator.py`** is the Composition Root.
* It owns the `pywebview` window, the API server thread, and all global services.
* It wires the `CommandBus` and `EventBus`.

### 3.2 Intent-Driven Interaction (The H-Pattern)

State changes must follow this path:
`Frontend UI` -> `POST /api/intents` -> `CommandBus` -> `Service Logic` -> `EventBus` -> `WebSocket` -> `Frontend Store`

* **Never** call services directly from API handlers. API handlers should only dispatch Intents.
* **Main/UI Thread**: Runs `pywebview` window (GTK on Linux, Cocoa on macOS, EdgeChromium on Windows)tly from the API layer.

### 3.3 Process & Threading Model

* **GTK/Main Thread**: Runs `pywebview`. **Zero blocking operations allowed here.**
* **API Thread**: Runs `Litestar` (async).
* **Recording Thread**: Audio capture + ASR inference via `faster-whisper` / CTranslate2 (in-process, background thread).
* **SLM Thread**: Text refinement runs in a background `Thread` via `ctranslate2` Generator.

### 3.4 Persistence Model

* Access persistence through **`TranscriptDB`**.
* **Immutability**: Original raw transcription is immutable.
* **Variants**: Edits and refinements are stored as variants linked to original transcript ID.

---

## 4. Repository Structure (Semantic and Binding)

* `frontend/` — Svelte 5 SPA. Independent build chain.
* `src/api/` — Litestar server definition and controllers.
* `src/core/` — Application plumbing (Coordinator, buses, settings, constants).
* `src/core/intents/` — Intent dataclass definitions.
* `src/services/` — Business logic (Audio, SLM, Transcription).

---

## 5. Critical Patterns

### 5.1 The API Boundary

The Python backend treats the Frontend as a remote client, even though they run locally.
* Communication via **REST** (for Actions) and **WebSocket** (for State Sync).
* API endpoints should be thin wrappers around `CommandBus.dispatch()`.

### 5.2 Svelte 5 & Runes

* Use **Svelte 5 Runes** (`$state`, `$derived`, `$effect`) for all reactivity.
* Do not use legacy Svelte 3/4 stores or `export let` syntax unless strictly necessary for library compatibility.
* Styling: Use **Tailwind CSS v4** utility classes. Avoid `<style>` blocks unless creating custom animations.

### 5.3 Service Isolation

* Services in `src/services/` must not import from `src/api/`.
* The API layer dispatches intents or reads data — it never reaches into service internals.

---

## 6. Coding Standards

### 6.1 Python

* Python **3.12+**.
* **Strict type hints** required.
* Use `pydantic` for data validation at the API boundary.

### 6.2 Frontend (TypeScript)

* Strict TypeScript mode.
* No `any` types. Define interfaces for all API payloads.
* Use `libs/api.ts` for backend communication.

---

## 7. Defensive Engineering Standards

### 7.1 Responsiveness

* **Strict Ban**: No blocking calls in `async def` API routes.
* **Strict Ban**: No blocking calls in the UI thread.
* Long-running tasks must offload to background threads/processes immediately.

### 7.2 Inference Safety


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [WanderingAstronomer/Vociferous](https://github.com/WanderingAstronomer/Vociferous) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
