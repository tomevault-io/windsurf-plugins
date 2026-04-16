---
trigger: always_on
description: Generates SQL strings from compiled types:
---

# CLAUDE.md — Ecrous Kernel

> This file is the complete architectural context for the Ecrous Kernel.
> Read it fully before making any code changes. Every section represents a deliberate decision.
> The codebase contains extensive `IMPL` comments that serve as implementation blueprints — read them before implementing anything.

---

## Plan Tracking

Before starting any multi-step work, create or update `docs/plans/{YYYY-MM-DD}-{slug}.md`.
Format:
- [ ] Step description
- [x] Completed step

After completing each step, update the checkbox. When resuming work (`claude --continue`), 
read the active plan file first to understand where we left off.

When a plan is fully complete, move it to `docs/plans/done/`.

---

## Project Identity

- **Project name:** Ecrous Kernel (part of the Ecrous platform)
- **Slogan:** *"Model once, derive everything"*
- **Repository:** `ecrous/kernel`
- **License:** AGPL-3.0 with output exception — generated artifacts (APIs, schemas, types, admin UIs) belong entirely to users. The engine itself is protected from proprietary forks. Dual licensing with commercial option available later.
- **Companion project:** Ecrous Studio (`ecrous/studio`) — visual model editor. Slogan: *"Your system, visually"*
- **Dev environment:** Nix flake with crane for Rust builds. `cargo-watch` and `rust-analyzer` in devShell.

---

## The Core Philosophy

Data models and their invariants are the single source of truth. Users define entities, fields, relationships, state machines, permissions, and invariants once — either through the Studio's visual editor or by writing YAML directly. The Kernel reads that model at startup and mechanically derives everything else: database schemas with constraints, REST API endpoints, TypeScript types, admin interfaces, migration plans.

**Key principle:** Correctness is enforced by construction across all layers, not by convention. If the model validates, the derived system is correct.

**Key invariant:** At request time, the kernel does ZERO interpretation of the model. Everything is pre-compiled into lookup tables and prepared statements. The only runtime costs are: HashMap lookups (~20ns), Postgres round-trips (~1-2ms), and JSON serialization (~10-50μs).

---

## Relationship to Studio

```
┌─────────────────────┐         model.yaml         ┌──────────────────┐
│                     │                             │                  │
│   Studio (Svelte)   │  ──── writes to disk ────▶  │  Kernel (Rust)   │
│                     │                             │                  │
│  Visual editor      │         hot reload          │  HTTP API server │
│  Canvas, panels     │  ◀──── watches file ─────   │  Postgres        │
│  Live preview       │                             │  Prepared SQL    │
│  Verification       │                             │                  │
└─────────────────────┘                             └──────────────────┘
```

- The Kernel does NOT need the Studio. You can write model YAML by hand.
- The Studio does NOT need the Kernel. It's a standalone editor.
- They connect through the **model file format** — that's the contract.
- Do NOT make the Kernel the backend for the Studio (circular dependency).

---

## Architecture Overview

```
YAML model file
      │
      ▼
  ┌──────────┐     ┌──────────────┐     ┌─────────────┐
  │  Parse   │ ──▶ │   Compile    │ ──▶ │   Verify    │
  │ (serde)  │     │ model→runtime│     │ cross-layer │
  └──────────┘     └──────────────┘     └─────────────┘
                         │
                         ▼
                  CompiledModel
                  ├─ EntityRegistry   (entity name → compiled entity)
                  ├─ RoutingTable     (method+path → handler)
                  ├─ PreparedSQL      (per entity, per operation)
                  └─ EventDispatcher  (event type → plugin handlers)
                         │
                         ▼
                   Kernel::serve()
                  (Axum HTTP server)
```

Three-stage pipeline: **Model** (what the user writes, YAML) → **Compiled** (optimized runtime structures) → **Served** (HTTP API backed by Postgres).

---

## Current State of the Code

**Everything is scaffolded, nothing is implemented.** Every method body is `todo!()`. The types and architecture are fully designed with extensive `IMPL` comments that serve as blueprints. The code compiles (type-checks) but does nothing at runtime.

### What exists:
- Complete type definitions for the model layer (6 layers of types)
- Complete type definitions for the compiled layer (runtime-optimized structures)
- Engine component shells with detailed pseudocode in comments
- Error type hierarchy (build-time and request-time errors)
- Kernel struct with ArcSwap-based hot reload architecture
- Nix flake dev environment

### What needs implementing (in order):
1. `ValidatorFactory::build()` — field type → validator closure
2. `SqlBuilder` methods — compiled types → SQL strings
3. `Compiler::compile()` — Model → CompiledModel
4. `Verifier::verify()` — structural checks (Level 1)
5. `Model::from_file()` / `Model::from_dir()` — YAML parsing
6. `Kernel::new()` — boot sequence
7. `Kernel::serve()` — Axum HTTP server with generic handlers
8. `Kernel::hot_reload()` — atomic model swap

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ecrous) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
