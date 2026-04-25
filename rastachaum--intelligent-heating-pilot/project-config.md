---
trigger: always_on
description: **This document is for direct Copilot interactions.** For team-based development:
---

# GitHub Copilot Instructions - Intelligent Heating Pilot (IHP)

## 📖 Using These Instructions

**This document is for direct Copilot interactions.** For team-based development:

- **See [`.github/agents/`](./agents/README.md)** for specialized agent roles and orchestration workflow
- **See [`docs/contributors/DEVELOPMENT_STANDARDS.md`](../docs/contributors/DEVELOPMENT_STANDARDS.md)** for practical development standards

## 🤖 Local Agent Invocation

When delegating work to specialized agents in this local environment:

- **Use `runSubagent` only** to invoke agents
- **Do NOT use `@developer`, `@qa-engineer`, or any `@...` mentions** (these do not work locally)

## 🎯 Project Overview

The Intelligent Heating Pilot (IHP) is a Home Assistant integration that intelligently preheats homes using predictive algorithms and machine learning. This document defines the architectural principles and development practices that **must** be followed by all AI-assisted code generation.

## 🛡️ Architectural Mandate: Domain-Driven Design (DDD)

All development must follow **Domain-Driven Design** principles with strict separation of concerns.

### Layer Structure

```
custom_components/intelligent_heating_pilot/
├── domain/              # Pure business logic (NO Home Assistant dependencies)
│   ├── value_objects/   # Immutable data carriers
│   ├── entities/        # Domain entities and aggregates
│   ├── interfaces/      # Abstract base classes (contracts)
│   └── services/        # Domain services
├── infrastructure/      # Home Assistant integration layer
│   ├── adapters/        # HA API implementations
│   └── repositories/    # Data persistence implementations
└── application/         # Orchestration and use cases
```

### Domain Layer Rules (⚠️ CRITICAL)

The **domain layer** contains the core intellectual property and must be completely isolated:

1. **NO Home Assistant imports** - Zero `homeassistant.*` imports allowed
2. **NO external service dependencies** - Only Python standard library and domain code
3. **Pure business logic** - If it models real-world heating behavior, it belongs here
4. **Interface-driven** - All external interactions via Abstract Base Classes (ABCs)
5. **Type hints required** - All functions and methods must have complete type annotations

### Infrastructure Layer Rules

The **infrastructure layer** bridges the domain to Home Assistant:

1. **Implements domain interfaces** - All adapters must implement ABCs from domain layer
2. **HA-specific code only** - All `homeassistant.*` imports belong here
3. **Thin adapters** - Minimal logic, just translation between HA and domain
4. **No business logic** - Delegate all decisions to domain layer

## 🧪 Test-Driven Development (TDD) + Behavior-Driven Development (BDD)

All new features must be developed using **Hybrid BDD/TDD strategy** to avoid redundancy and maximize clarity.

**⚠️ MANDATORY**: Agents MUST follow the comprehensive testing strategy defined in [`agents/TESTING_STRATEGY.md`](./agents/TESTING_STRATEGY.md)

### Quick Decision Guide

**Use Pytest-BDD (Gherkin)** for:
- Business-observable behavior (Black Box)
- Happy paths and user scenarios
- Features a Product Owner can understand
- High-level interaction validation

**Use Pytest Unitaires (TDD)** for:
- Edge cases (None, empty, overflow)
- Exception handling (errors, timeouts, failures)
- Algorithmic correctness (calculations, FIFO, sorting)
- Technical robustness (type validation, memory limits)

**❌ DO NOT DUPLICATE**: If a happy path is covered by BDD, do NOT create equivalent unit test (unless type/performance validation is required).

### BDD: Gherkin Feature Files

Write business scenarios in `tests/features/*.feature`:

```gherkin
Feature: Heating Cycle Cache Management
  Scenario: Cache stores LHS slope after heating cycle
    Given a heating cycle is running
    When the cycle completes
    Then cache stores the measured LHS slope
    And next preheating uses the cached slope
```

Convert to pytest using **pytest-bdd** fixtures and step definitions.

### TDD: Unit Testing Requirements

1. **Domain-first testing** - Write domain layer tests BEFORE implementation
2. **Mock external dependencies** - Use mocks for all infrastructure interactions
3. **Test against interfaces** - Unit tests should test against ABCs, not concrete implementations
4. **Centralized fixtures** - Use a centralized `fixtures.py` file for test data (DRY principle)
5. **High coverage** - Aim for >80% coverage of domain logic
6. **Fast tests** - Domain tests should run in milliseconds (no HA, no I/O)
7. **Non-redundancy** - Do NOT create unit tests for scenarios already covered by BDD

### Testing Structure

```python
tests/
├── features/            # BDD acceptance criteria (Gherkin)
│   ├── heating_cache.feature
│   └── conftest.py      # pytest-bdd fixtures and step definitions
├── unit/
│   ├── domain/          # Pure domain logic tests (no mocks needed for value objects)
│   │   ├── test_value_objects.py
│   │   ├── test_pilot_controller.py
│   │   └── test_domain_services.py
│   └── infrastructure/  # Adapter tests (with mocked HA)
│       ├── test_scheduler_reader.py
│       └── test_climate_commander.py
└── integration/         # End-to-end tests (optional, slower)
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RastaChaum/Intelligent-Heating-Pilot](https://github.com/RastaChaum/Intelligent-Heating-Pilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
