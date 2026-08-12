---
trigger: always_on
description: Irihi.Mafia is an Avalonia component library and theme system for **mobile-first** applications.
---

# Irihi.Mafia Copilot Instructions

## Project Goal

Irihi.Mafia is an Avalonia component library and theme system for **mobile-first** applications.

The target visual and interaction reference is **Tencent TDesign Mobile Vue** rather than generic TDesign desktop components. When there is uncertainty, prefer the behavior, naming, examples, and interaction patterns from the **`mobile-vue`** component set.

This repository is mainly used to build:

1. Reusable controls in `src/Irihi.Mafia`
2. A TDesign Mobile-inspired theme implementation in `src/Irihi.Mafia.Themes.TDesign`
3. Demo experiences in `demo/` for phone-sized layouts and mobile scenarios
4. Unit and headless UI tests in `test/`

## Official Reference Sources

Use these references in priority order depending on the question being answered:

1. **TDesign Mobile Vue MCP** for component semantics, API naming, examples, and mobile interaction patterns
2. **Avalonia Build MCP** for official Avalonia docs, API reference, and framework-specific implementation rules
3. **Semi.Avalonia** as a local implementation reference for how another Avalonia component library structures themes, controls, and styling patterns when those patterns fit this repository
4. **Ursa.Avalonia** as an additional Avalonia implementation reference for control architecture, theme organization, and library-level patterns when those patterns fit this repository

Reference link for Semi.Avalonia:

- https://github.com/irihitech/Semi.Avalonia

Reference link for Ursa.Avalonia:

- https://github.com/irihitech/Ursa.Avalonia

When using the TDesign MCP tools:

1. Prefer the **`mobile-vue`** framework
2. Check component docs before inventing API or interaction details
3. Use component demo and DOM references when translating structure or states
4. Treat TDesign Mobile Vue as the source of truth for mobile semantics, not desktop TDesign

When using the Avalonia Build MCP tools:

1. Use `get_avalonia_expert_rules` early in a session when implementing or refactoring controls
2. Use `search_avalonia_docs` and `lookup_avalonia_api` before guessing Avalonia API behavior
3. Prefer official Avalonia guidance over generic XAML or WPF assumptions

When using Semi.Avalonia as a reference:

1. Reuse architectural ideas and Avalonia-specific implementation patterns, not Semi branding or API names
2. Prefer Semi.Avalonia for internal structure inspiration only when it does not conflict with TDesign Mobile Vue semantics
3. Do not copy behavior that would push the component toward desktop-first interaction

When using Ursa.Avalonia as a reference:

1. Reuse architecture, project organization, and Avalonia implementation patterns that fit this repository
2. Prefer Ursa.Avalonia as a secondary structural reference, not as the product semantics source
3. Do not copy UX behavior that conflicts with TDesign Mobile Vue mobile-first expectations

## Repository Structure

- `src/Irihi.Mafia/`
  - Core controls and shared runtime code
  - Custom controls belong here when they introduce public API or reusable behavior
- `src/Irihi.Mafia.Themes.TDesign/`
  - TDesign Mobile-inspired styling and resources
  - `Controls/`: control themes and previews
  - `Themes/Shared`: structure and logic shared across light and dark
  - `Themes/Light` and `Themes/Dark`: theme-specific resource overrides
  - `Tokens/`: design tokens and semantic resources
- `demo/Irihi.Mafia.Demo/`
  - Demo views and usage examples
  - New public components should have a mobile-oriented example
- `test/Irihi.Mafia.UnitTest/`
  - Logic and API tests
- `test/Irihi.Mafia.HeadlessTest/`
  - Avalonia headless UI tests

## Working Rules

Keep the existing separation of concerns:

1. Put reusable control API in `src/Irihi.Mafia`
2. Put visual styling in `src/Irihi.Mafia.Themes.TDesign`
3. Put usage examples in `demo/`
4. Put behavior verification in `test/`

Do not implement a component only in the demo or only in theme files if it needs reusable public API.

## Additional Reference Guidance

- If a component behavior question is about **mobile product semantics**, follow TDesign Mobile Vue first
- If a question is about **Avalonia mechanics**, follow Avalonia Build MCP first
- If a question is about **how to organize Avalonia theme/control code**, Semi.Avalonia can be used as a secondary structural reference
- When these sources disagree, prefer:
  1. TDesign Mobile Vue for UX semantics
  2. Avalonia official docs for framework correctness
  3. Semi.Avalonia for implementation inspiration

## Mobile-First Control Conventions

### Public API

- Prefer Avalonia styled properties for configurable control state
- Keep property names PascalCase
- Keep nullable reference types enabled
- Reuse Avalonia concepts where possible, but align semantic concepts with **TDesign Mobile Vue**
- Keep mobile concepts explicit when needed, such as `Placement`, `PopupVisible`, `SafeAreaInset`, `Size`, `Status`, `Orientation`, or `Trigger`

### Interaction

- Optimize for touch interaction over pointer-precision interaction
- Prefer larger hit targets and clear pressed / disabled / loading feedback
- Consider safe areas for bottom bars, action sheets, dialogs, and full-screen pages

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [irihitech/Irihi.Mafia](https://github.com/irihitech/Irihi.Mafia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
