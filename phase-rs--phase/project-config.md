---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

phase.rs is a Magic: The Gathering game engine written in Rust (compiling to native + WASM) with a React/TypeScript frontend. It implements MTG game rules using functional architecture (discriminated unions, pure reducers, immutable state) with an Arena-quality browser UI. Card data is sourced from MTGJSON (MIT-licensed) with custom typed JSON ability definitions.

## Design Principles — READ THIS FIRST

**Above all else, this project prioritizes three co-equal pillars: idiomatic Rust, composable building-block architecture, and strict fidelity to the MTG Comprehensive Rules. These are non-negotiable and override convenience, speed-of-delivery, or "getting it working." Every code change must pass through all three lenses before anything else.**

- **Idiomatic Rust, always.** Use Rust's type system, ownership model, and idioms to their fullest. Prefer `enum` over stringly-typed data. Prefer exhaustive `match` over fallback defaults. Prefer trait-based polymorphism over dynamic dispatch when the type set is known. If the idiomatic path is harder, take it anyway — shortcuts compound into debt.
- **Rules-correct over convenient — the #1 hard rule.** This is an MTG rules engine — correctness to the Comprehensive Rules is a hard requirement, not a nice-to-have. Every implementation pattern MUST be verified against the relevant CR section before it is considered complete. When a rules-correct implementation is more complex than a shortcut, take the complex path. A simpler implementation that gets the rules wrong is not simpler — it is wrong. If you are unsure whether a behavior is rules-correct, look up the CR section, annotate the code, and implement what the rules say, not what seems reasonable. "It works for most cases" is not acceptable when the CR specifies exact behavior. No game logic ships without CR validation.
- **Build for the class, not the card.** Every new enum variant, parser pattern, effect handler, or filter must handle a *category* of cards, not a single card. Before writing any logic, ask: "How many cards does this cover?" If the answer is one, you're building a special case — find the general pattern and build that instead. A one-off that works for one card but breaks for the next card with the same pattern is not a building block; it is technical debt.
- **The engine owns all logic.** All game rules, validation, derived state, and computed values live in the `engine` crate. Transport layers (WASM bridge, Tauri IPC, WebSocket server) are thin serialization boundaries — zero game logic allowed. If multiple consumers need the same behavior, it belongs in the engine. Never duplicate logic across adapters. When in doubt, put it in the engine.
- **The frontend is a display layer, not a logic layer.** The React client renders engine-provided state and dispatches user actions — nothing more. It must never compute, derive, transform, or re-interpret game data. If the frontend needs a value, the engine must provide it. Formatting for display (e.g., string interpolation of engine-provided fields) is acceptable; calculating, filtering, or inferring game state is not. Any "smart" frontend code is a bug — move it to the engine.
- **Compose from building blocks.** Every new capability should be decomposed into reusable primitives that unlock future features. Before writing specific logic, ask: "What is the general pattern here?" and build that instead. This applies equally to data modeling: when a new field or parameter needs to distinguish cases, use an existing typed enum (e.g., `ControllerRef`, `Comparator`, `Option<T>`) — never a raw `bool`. A boolean isn't composable; an existing type is self-documenting, extensible, and expresses the full design space. Examples: `contains_possessive`/`contains_object_pronoun` for Oracle text matching, `ChangeZone` + `Shuffle` composition for compound shuffles, `Option<ControllerRef>` for "whose turn is required" instead of `requires_your_turn: bool`.
- **Nom combinators on the first pass — no exceptions.** All new parser code MUST use nom combinators (`tag()`, `alt()`, `value()`, `terminated()`, `pair()`, etc.) from the very first line written. Never write `find()`, `split_once()`, `contains()`, or `starts_with()` for parsing dispatch and then "plan to convert to combinators later." There is no later — write it correctly the first time. Use `nom_on_lower` bridge for mixed-case text, `tag().parse()` for already-lowercase text. Use existing building blocks (`parse_single_cost`, `parse_target`, `parse_for_each_clause`, etc.) for composed operations. If you catch yourself writing string matching for parsing, stop and rewrite with combinators before proceeding. This has been a recurring issue and is non-negotiable.
- **Extend, don't hack.** New features should slot cleanly into existing patterns (effect handlers, game modules, ability definitions). If a feature requires working around the architecture, the architecture should be extended first.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [phase-rs/phase](https://github.com/phase-rs/phase) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
