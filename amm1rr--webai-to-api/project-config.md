---
trigger: always_on
description: This document specifies the architectural governance, runtime contracts, and operational guidelines for WebAI-to-API. It is the authoritative guide for contributors and AI agents working on the browser-native runtime.
---

# WebAI-to-API: Agent & Contributor Guide

This document specifies the architectural governance, runtime contracts, and operational guidelines for WebAI-to-API. It is the authoritative guide for contributors and AI agents working on the browser-native runtime.

> **Scope of this Document**: This file serves as a contributor governance guide. Detailed behavioral guarantees and technical invariants are codified in the [Runtime Contract Map](#6-runtime-contract-map) inside `docs/`, which remain the normative authority for the system. The runtime contracts primarily govern the hardened Playwright runtime layer.

## Mandatory Architectural Directives

* **Architectural Consistency:** Preserve the established runtime architecture, ownership boundaries, and lifecycle invariants.
* **Clean & Maintainable Code:** Prefer clarity, determinism, and maintainability over clever or overly abstract solutions.
* **No Unscoped Refactors:** Avoid unrelated rewrites or architectural changes unless explicitly required.
* **Documentation Consistency:** Keep comments, structured logs, and documentation aligned with the implemented behavior whenever modifying logic.
* **Pragmatic Engineering:** Avoid unnecessary over-engineering. Implement the smallest solution that fully satisfies the runtime contracts and operational requirements.
* **Regression Safety:** New changes must preserve existing runtime guarantees, API behavior, and operational stability unless an intentional breaking change is explicitly requested.
* **Evidence-Driven Changes:** Base technical decisions on actual code paths, runtime behavior, and authoritative specifications rather than assumptions.
* **Structural Discipline:** Follow the established project structure, ownership hierarchy, and authoritative runtime contracts in `docs/`.

## 1. Technical Vision

**WebAI-to-API** is a specialized **Web AI Runtime** that converts browser-based AI interfaces into high-availability, OpenAI-compatible APIs. Unlike legacy systems that rely on reverse-engineered internal protocols, this runtime drives real browser sessions via Playwright, ensuring strong resilience against web UI updates and providing a deterministic and strongly-governed runtime layer for browser-native LLM integration.

---

## 2. Core Runtime Architecture

The system operates according to a strict ownership hierarchy and state machine.

### 2.1 Component Hierarchy
1. **BrowserEngine**: Global singleton. Authoritative orchestrator for the core Chromium process and coordinator for terminal shutdown.
2. **ProviderSession**: Created per provider (Gemini, etc.). Authoritative owner of the `BrowserContext`, the `keepalive_page`, and session-scoped recovery logic (context recreation, tab invalidation).
3. **AuthManager**: Provider-agnostic orchestrator for authentication lifecycle, concurrency locks, and background login tasks.
4. **ManagedPage**: Request-scoped resource container. Authoritatively owns exactly one semaphore permit and one `PersistentTab` lease.
5. **PersistentTab**: Long-lived browser page in the session registry. Owns its individual `_lock` and internal state.

### 2.2 Lifecycle & Ownership
- **Generation Invalidation**: Tracks browser process generations to automatically invalidate stale contexts, `PersistentTab` objects, active leases, cached references, and request-scoped bridge state after a browser generation rollover or fatal disconnect.
- **Terminal Shutdown**: Once `BrowserEngine` initiates shutdown, the active engine lifecycle cannot be resurrected. Runtime components must fail fast and may never re-initialize the browser after terminal shutdown begins.
- **Deterministic Teardown**: Request cleanup MUST follow a strict sequence (Observers -> Tasks -> Callbacks -> Queues -> Leases) to prevent late-event races.
- **Auth Separation**: Provider-specific authentication logic (URLs, selectors, recovery hooks) is encapsulated in dedicated strategy classes (e.g., `GeminiAuthStrategy`) and registered with the `AuthManager`.

---

## 3. Concurrency & Locking

### 3.1 Lock Hierarchy
To prevent deadlocks, locks must be acquired strictly in this order. Acquiring out-of-order is strictly forbidden:
1. `BrowserEngine.management_lock` (Global orchestration)
2. `ProviderSession.init_lock` (Session setup/recovery)
3. `ProviderSession.registry_lock` (Synchronous registry mutations only)
4. `PersistentTab._lock` (Individual tab operations)

**Discipline**: `registry_lock` must NEVER be held across `await` points or long-running Playwright operations. Violating this scope discipline risks global request starvation.

### 3.2 Semaphore & Lease Semantics
- **Exclusivity**: Active browser operations require a valid lease on a `PersistentTab`.
- **Mandatory Shielding**: Resource release (locks, permits, registry entries) must be performed via `asyncio.shield` to guarantee completion during request cancellation.
- **Idempotency & Best-Effort**: Cleanup paths and `ManagedPage.close()` must be idempotent and best-effort. Failures in one step must not block subsequent resource release.

---

## 4. Provider & Streaming Contracts

### 4.1 Required Hooks

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Amm1rr/WebAI-to-API](https://github.com/Amm1rr/WebAI-to-API) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
