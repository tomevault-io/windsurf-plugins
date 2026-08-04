---
trigger: always_on
description: This document defines the architecture, naming conventions, and rules that all contributors and AI agents must follow when working on this codebase.
---

# AGENTS.md — Architecture & Development Rules

This document defines the architecture, naming conventions, and rules that all contributors and AI agents must follow when working on this codebase.

---

## Project Overview

`homebridge-securitysystem` is a Homebridge v2 accessory plugin (not a platform plugin) that exposes a fully-featured security system to HomeKit. It is written in TypeScript with ESM modules.

The plugin:
- Exposes one `SecuritySystem` HAP service plus up to 22 optional switch/sensor accessories.
- Uses an event-driven architecture: core state changes emit domain events; side-effect services (audio, webhook, command) listen and react.
- Uses an abstract `Condition` class hierarchy to encapsulate all blocking-logic decisions.
- Provides an optional Hono HTTP server for remote control.

---

## Source Layout

```
src/
  @types/            Third-party type declarations
  conditions/        Abstract Condition base + concrete condition classes
  constants/         Compile-time constants (no logic)
  handlers/          Stateful handlers wired by security-system.ts
  interfaces/        TypeScript interfaces (plain object shapes)
  services/          Stateful services (audio, webhook, command, storage, server, event bus)
  tests/             Vitest test suites
  types/             TypeScript enums and type aliases
  utils/             Pure utility functions
  index.ts           Homebridge plugin entry point
  security-system.ts Root AccessoryPlugin class
```

---

## Layer Rules

| Layer | What belongs here | What does NOT belong |
|---|---|---|
| `types/` | Enums, type aliases | Logic, classes |
| `interfaces/` | Plain object shape interfaces | Logic, classes, enums |
| `constants/` | `const` objects, literal values | Logic, mutable state |
| `utils/` | Pure functions with no side effects | Classes, state, I/O |
| `conditions/` | Classes extending `Condition` | Handlers, services |
| `services/` | Stateful singleton classes, I/O | Embedded types, enums, interfaces |
| `handlers/` | State-machine logic classes | Embedded types, enums, interfaces |

**Types, enums, and interfaces must never be defined inside a service or handler file.** Always create a separate file in `types/` or `interfaces/` and import it.

---

## File Naming Conventions

All filenames use **kebab-case** with a mandatory suffix describing their kind:

| Kind | Suffix | Example |
|---|---|---|
| Enum or type alias | `-type.ts` | `security-state-type.ts` |
| Interface | `-interface.ts` | `system-state-interface.ts` |
| Constant object | `-constant.ts` | `homekit-constant.ts` |
| Utility functions | `-util.ts` | `state-util.ts` |
| Service class | `-service.ts` | `audio-service.ts` |
| Handler class | `-handler.ts` | `state-handler.ts` |
| Condition class | `-condition.ts` | `double-knock-condition.ts` |
| Test suite | `.test.ts` | `conditions.test.ts` |

---

## Naming Conventions

### No Abbreviations

**Variable names, parameter names, and type names must not use abbreviations.**

This rule applies everywhere: source files, test files, and any new code.

Forbidden examples and their correct replacements:

| Forbidden | Use instead |
|---|---|
| `s`, `st` for state | `state` |
| `o`, `opts` for options | `options` |
| `v` for value | `value` |
| `req` for request | `request` |
| `res` for response | `response` |
| `c` for context | `context` |
| `e` for error | `error` |
| `cb` for callback | `callback` |
| `fn` for function | the actual semantic name |
| `svc` for service | `service` |
| `char` for characteristic | `characteristic` |
| `Char` for Characteristic constructor | `Characteristic` |
| `Svc` for Service constructor | `Service` |
| `proc` for process | `process` |
| `dir` for directory | `directory` |
| `msg` for message | `message` |
| `buf` for buffer | `buffer` |
| `idx` for index | `index` |
| `len` for length | `length` |
| `num` for number | `number` or a semantic name |
| `str` for string | the semantic name |
| `tmp` for temporary | the semantic name |
| `args` for arguments | `arguments` (or a semantic name) |

**Exception:** loop variables `i`, `j`, `k` in tight numeric loops where the name carries no domain meaning are acceptable. All other names must be descriptive.

### Class Names

- PascalCase, no abbreviations.
- Suffix matches the layer: `...Service`, `...Handler`, `...Condition`.

### Enum Members

- UPPER_SNAKE_CASE (e.g. `SecurityState.TRIGGERED`).

### Interface Names

- PascalCase, no `I` prefix.
- Suffix: `...Interface` is not used in the filename's export — the filename carries it.

---

## Architecture: Event-Driven Side Effects

The core state machine (`StateHandler`, `TripHandler`, `SwitchHandler`) never calls side-effect services directly. Instead it emits domain events via `EventBusService`:

```
StateHandler → bus.emit(EventType.CURRENT_CHANGED, payload)
                         ↓              ↓              ↓
                  AudioService   WebhookService  CommandService
```

Each side-effect service calls `attachToBus(bus)` during construction in `security-system.ts`.

**New side effects must follow this pattern** — never add direct calls from handlers to services.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MiguelRipoll23/homebridge-securitysystem](https://github.com/MiguelRipoll23/homebridge-securitysystem) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
