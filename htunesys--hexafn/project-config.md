---
trigger: always_on
description: SPDX-FileCopyrightText: 2025 Husamettin ARABACI
---

<!--
SPDX-FileCopyrightText: 2025 Husamettin ARABACI
SPDX-License-Identifier: MIT
-->

# GitHub Copilot Instructions for hexaFn

## ⚡ PROJECT OVERVIEW

**hexaFn** is a modular, event-driven function pipeline framework powered by the **6F Lifecycle Flow**: **Feed → Filter → Format → Function → Forward → Feedback**. This project follows **Hexagonal Architecture (Ports & Adapters)** with **Domain-Driven Design (DDD)** principles.

**Company**: hexaTune LLC  
**Maintainer**: Husamettin ARABACI (@husamettinarabaci)  
**License**: MIT (SPDX-compliant, REUSE Spec 3.3 compatible)  
**Repository**: <https://github.com/hTuneSys/hexaFn>

---

## 🚨 CRITICAL ARCHITECTURE REQUIREMENTS

### Hexagonal Architecture Structure (for all modules EXCEPT hexafn-core and hexafn-macros)

```plaintext
crates/<domain>/src/
├── domain/              # Core business rules (entities, value objects, domain services)
│   ├── entities/        # Aggregates and core business objects with identity
│   ├── value_objects/   # Immutable value types without identity
│   ├── events/          # Domain events for cross-domain communication
│   ├── services/        # Domain services containing business logic
│   └── contracts/       # Domain contracts and interfaces
├── application/         # Use cases, application services, port definitions
│   ├── commands/        # State-changing operations (CQRS Commands)
│   ├── queries/         # Read operations (CQRS Queries)
│   ├── ports/           # Interface definitions (dependency inversion)
│   └── services/        # Application orchestration services
├── infrastructure/      # Concrete implementations of output ports
│   ├── persistence/     # Database adapters, repositories
│   ├── messaging/       # Event publishing, message queue adapters
│   └── external/        # External API clients, third-party integrations
└── lib.rs               # Crate entry point (re-exports and public API)
```

### 6F Lifecycle Flow Integration

1. **Feed**: Ingest from external sources (events, APIs, queues)
2. **Filter**: Pre-condition checks and gating
3. **Format**: Normalize, transform, validate
4. **Function**: Execute logic with user-defined behavior
5. **Forward**: Route results to KV stores, topics, services
6. **Feedback**: Log, trace, trigger, or audit

### Module Boundaries

- `hexafn-core`: Core architecture & event flow ( Shared Kernel )
- `hexafn-macros`: Procedural macro crate (attribute macros, codegen, DSL registration)
- `hexafn-bridge`: HexaBridge (external integrations & webhooks)
- `hexafn-trigger`: HexaTrigger (conditional logic & rule engine)
- `hexafn-run`: HexaRun (function runtime: WASM, JS, DSL)
- `hexafn-store`: HexaStore (typed KV storage with triggers)
- `hexafn-cast`: HexaCast (pub-sub messaging engine)
- `hexafn-watch`: HexaWatch (observability & audit tracing)

---

## 🚨 MANDATORY DOCUMENTATION REFERENCE

- [`docs/TODO_LIST.md`](../docs/TODO_LIST.md) - **MANDATORY** for task tracking and sprint planning
- [`docs/DATA_MODEL_CORE.puml`](../docs/DATA_MODEL_CORE.puml) - **MANDATORY** Data Core model and domain concepts
- [`docs/DATA_MODEL_BRIDGE.puml`](../docs/DATA_MODEL_BRIDGE.puml) - **MANDATORY** Data Bridge model and domain concepts
- [`docs/DATA_MODEL_TRIGGER.puml`](../docs/DATA_MODEL_TRIGGER.puml) - **MANDATORY** Data Trigger model and domain concepts
- [`docs/DATA_MODEL_RUN.puml`](../docs/DATA_MODEL_RUN.puml) - **MANDATORY** Data Run model and domain concepts
- [`docs/DATA_MODEL_STORE.puml`](../docs/DATA_MODEL_STORE.puml) - **MANDATORY** Data Store model and domain concepts
- [`docs/DATA_MODEL_CAST.puml`](../docs/DATA_MODEL_CAST.puml) - **MANDATORY** Data Cast model and domain concepts
- [`docs/DATA_MODEL_WATCH.puml`](../docs/DATA_MODEL_WATCH.puml) - **MANDATORY** Data Watch model and domain concepts
- [`docs/DATA_FLOW.md`](../docs/DATA_FLOW.md) - **MANDATORY** Data flow patterns
- [`docs/DATA_FLOW_SAMPLE.md`](../docs/DATA_FLOW_SAMPLE.md) - **MANDATORY** Data flow patterns sample
- [`docs/DATA_FLOW_DETAIL.md`](../docs/DATA_FLOW_DETAIL.md) - **MANDATORY** Detailed component architecture
- [`docs/BRANCH_STRATEGY.md`](../docs/BRANCH_STRATEGY.md) - **MANDATORY** for branch naming and workflow
- [`docs/COMMIT_STRATEGY.md`](../docs/COMMIT_STRATEGY.md) - **MANDATORY** for commit conventions
- [`docs/PR_STRATEGY.md`](../docs/PR_STRATEGY.md) - **MANDATORY** for pull request process
- [`docs/LABELLING_STRATEGY.md`](../docs/LABELLING_STRATEGY.md) - **MANDATORY** for issue/PR labeling

---

## 🚨 MANDATORY COMMIT & BRANCH RULES

### **REQUIRED**: Commit Message Format

```text
<type>(optional-scope): <short summary>
```

### **REQUIRED**: Allowed Commit Types ONLY

- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation only changes
- `style`: Code style, formatting (no logic change)
- `refactor`: Code refactoring (no feature change)
- `perf`: Performance improvements
- `test`: Adding or modifying tests
- `ci`: Changes to CI/CD configuration
- `build`: Build system or dependencies
- `chore`: Maintenance or tooling
- `release`: Versioning, changelog updates
- `hotfix`: Emergency fix for critical issues

### **REQUIRED**: Branch Naming Convention

Branch names MUST follow: `<type>/<description>`

If there is a related issue ID, it should be appended at the end: `<type>/<description>-<issueID>`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hTuneSys/hexaFn](https://github.com/hTuneSys/hexaFn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
