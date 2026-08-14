---
trigger: always_on
description: ReignsAgent is a production-oriented, Reigns-like agentic game product for generating, testing, editing, previewing, and shipping card-based narrative experiences.
---

# ReignsAgent System Specification & Agent Guidelines

## 0. Project Goal
ReignsAgent is a production-oriented, Reigns-like agentic game product for generating, testing, editing, previewing, and shipping card-based narrative experiences.

This is not only a developer middleware project. The final system must be usable by real content creators through a functional frontend where they can import content, configure or connect AI generation, edit generated cards, structure narrative progression, preview gameplay, inspect narrative and balance diagnostics, and prepare deployable game builds. Architecture must remain separated, but the product goal includes a complete user-facing creation and publishing workflow.

The long-term goal is a self-improving loop:
- **Core** runs the game rules headlessly and stays small, deterministic, and UI-free.
- **Reviewer** stress-tests card sets through Monte Carlo simulation and emits objective JSON diagnostics for coverage, pacing, endings, dead paths, and balance.
- **Pipeline** imports local content, calls generation connectors, and uses Reviewer diagnostics to propose or generate corrections.
- **Interface** provides the production frontend for content development, AI configuration, editing, preview, diagnostics, and deployment preparation while preserving module boundaries.

The project should optimize for pure left/right card interaction, inspectable data, automated narrative/balance feedback, and modular evolution. The player-facing game must remain the cleanest possible Reigns-style swipe experience, but the authored content may still contain data-driven chapters, themes, story arcs, endings, and stateful narrative evolution.

Narrative progression and Anti-RPG boundary: the system may support data-driven story progression through author-defined tags, variables, metadata, chapters, themes, arcs, endings, and configurable state/gauge presentation. These are narrative organization and authoring concepts, not built-in RPG management systems. The system must not ship predefined equipment, pets, inventory, shop, rarity, crafting, character-build, skill-tree, loot, or resource-management gameplay. Any such concepts may exist only as user-authored data labels interpreted by custom content, not as built-in gameplay loops, UI systems, or product features. The visible gameplay remains pure card text plus binary left/right choices.

## 1. Core Architecture (Decoupled Design)
- **Module A: ReignsAgent-Core**: Headless runtime. Handles default four gauge/faction values (0-100), loops, game-over, variable/tag state, and card scheduler. No UI/IO, no AI, and no hard-coded chapter or RPG systems.
- **Module B: ReignsAgent-Pipeline**: Handles local JSON/CSV import/export and LLM API connectors for automated content & asset generation.
- **Module C: ReignsAgent-Reviewer**: Independent Monte Carlo simulation engine. Runs 100k cycles headlessly, generates JSON diagnostic reports for narrative coverage and balance, and feeds back to Module B for self-correction.

## 2. Extensibility Specification
- **Variable Store**: Must support a dynamic low-level variable/tag store for user-authored content state, including narrative progression state.
- **Narrative Organization Labels**: Content authors may define chapters, themes, arcs, endings, status labels, and gauge presentation through data and metadata. These labels must remain author-owned and data-driven unless a later reviewed schema explicitly promotes them.
- **Custom Data Labels**: Content authors may use arbitrary labels in their own data, but the engine must not ship predefined equipment, pet, shop, rarity, crafting, class, skill-tree, loot, or inventory-management concepts.
- **Lifecycle Hooks**: Variable/tag hooks may implement `on_acquire`, `on_tick`, and `on_dismiss` to modify card pool weights and faction scales dynamically. These hooks are engine-level extension points only, not a mandate to build item systems or UI.
- **AI Assist Boundary**: AI Assist is a creator-side workflow for request planning, contextual draft generation, review repair, visual request preview, and patch proposal application. Core and deployable player builds must not contain provider SDKs, API keys, network AI calls, generated-edit tooling, or AI-specific gameplay behavior. AI output must flow through explicit proposals, editor validation, player validation where relevant, and undo/draft history before it becomes authored content.
- **AI Endpoint UX**: The default creator UX should prefer lightweight user-supplied endpoint configuration (base URL, API key, protocol, model id, and capability flags) over heavy provider-profile management. Multiple profiles, provider presets, model discovery, MCP, skills, and arbitrary agent tools are optional developer-mode enhancements unless a later reviewed plan promotes them.
- **Desktop Host Boundary**: Electron is an optional outer host only. `apps/desktop-electron` may start the shared Creator Server and load the compiled WebUI, but Core, Pipeline, Reviewer, Interface, Creator Web, and Creator Server must not import Electron or depend on desktop-only APIs.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Sisyphe42/ReignsAgent](https://github.com/Sisyphe42/ReignsAgent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
