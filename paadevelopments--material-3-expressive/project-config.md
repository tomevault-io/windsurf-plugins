---
trigger: always_on
description: > These rules apply to **every AI agent** (Copilot, Cursor, Gemini, Claude, etc.) that touches this repository.
---

# AGENTS.md — Contributor Rules for AI Agents

> These rules apply to **every AI agent** (Copilot, Cursor, Gemini, Claude, etc.) that touches this repository.
> Read this file completely before writing a single line of code.

This document complements **The base rule** in [`.cursor/rules/base-rule.mdc`](.cursor/rules/base-rule.mdc). When both apply, follow both; if anything conflicts, treat the base rule as the tighter project-specific constraint.

---

## 1. Understand Before You Act

**Every session must begin with a project orientation pass.** Before any implementation or fix, you MUST review:

- **Project structure**: [`lib/components/`](lib/components/) — one folder per component (`buttons/`, `sliders/`, `date_pickers/`, …); [`lib/foundations/`](lib/foundations/) — shared theme, tokens, motion, tappable, and forthcoming haptics
- **Public API surface**: [`lib/material_3_expressive.dart`](lib/material_3_expressive.dart) is the single export barrel — understand what is and is not exported
- **Architecture patterns**: Each component follows the pattern in §2 — understand it before adding to it
- **Example app**: [`example/lib/`](example/lib/) — run it and trace your component’s demo **page** before making changes
- **Changelog**: [`CHANGELOG.md`](CHANGELOG.md) — understand what has recently changed and why
- **README**: [`README.md`](README.md) — understand the user-facing surface and component documentation

Do not assume you remember the structure from a previous session. Re-read what is relevant each time.

---

## 2. Project Architecture

This is a **Flutter UI component library** implementing Material 3 Expressive (M3E) components.

### Top-level layout

```
lib/
├── material_3_expressive.dart   # public barrel
├── components/                  # ~39 component modules
│   └── <component>/
└── foundations/                 # theme, tokens, shared primitives
```

There is **no** `lib/src/`.

### Component directory pattern

Each component lives under `lib/components/<component-name>/` and may contain (only if needed):

```
lib/components/<component>/
├── m3e_<component>.dart    # entry file(s); may hold variations
├── components/             # sub-widgets
├── enums/
├── styles/                 # theme / token data classes
├── models/
├── controllers/
├── utils/
└── res/                    # static constants (not enums)
```

**Example — `buttons/`:**

```
lib/components/buttons/
├── m3e_buttons.dart
├── components/
├── enums/
├── styles/
└── ...
```

### Foundations

[`lib/foundations/`](lib/foundations/) holds design tokens, theming, shapes, motion, and shared interaction primitives. Key exports are wired through [`lib/foundations/foundations.dart`](lib/foundations/foundations.dart).

Subfolders (import via the barrel unless you are editing foundations internals):

- `color/` — color scheme and utilities
- `shape/` — shapes, clippers, `material_new_shapes` bridge
- `theme/` — `M3ETheme`, `M3EThemeData`, `M3EMaterialApp`, dynamic color host
- `type/` — typography, type-style tokens, variable-font config
- `interaction/` — motion, haptics, tappable, state layers, ink splash
- `tokens/` — spacing, elevation, dimensions, safe area, scrim
- `components/` — shared `M3EComponentTheme`
- `m3e_icons.dart` — generated glyph dump (root only)

Notable areas:

- **Theme**: `M3ETheme`, `M3EThemeData`, `M3EThemeScope`, `M3EMaterialApp`
- **Color**: `M3EColorScheme`, `M3EDynamicColorHost`
- **Tokens**: typography, spacing, elevation, shapes, state layers
- **Motion / interaction**: `m3e_motion.dart`, `M3ETappable`
- **Shapes bridge**: `m3e_material_new_shapes_bridge.dart` → `material_new_shapes`
- **Haptics (forthcoming)**: shared expressive haptics will live here (see §2.1)

### 2.1 Haptics (forthcoming)

The package **will add** a shared haptics implementation under **foundations** (same layer as motion and `M3ETappable`), for example `lib/foundations/interaction/m3e_haptics.dart`, exported via `foundations.dart` and the public barrel when ready.

Rules for agents:

- Use the **shared M3E haptics API** for tactile feedback — do **not** scatter raw `HapticFeedback.*` or one-off platform calls across components.
- If the shared API does not cover a case, **ask** before inventing a parallel path.
- Pair haptics with spring/expressive motion where the M3E spec calls for feedback (press, select, snap, dismiss, etc.).
- Respect platform no-ops / reduced capability (same spirit as `dynamic_color` on unsupported platforms).

### Key design principles

- **Spring physics first**: Animations use the `motor` package for spring-driven motion (not Flutter `AnimationController` curves directly unless there is a strong reason). Foundations motion and `M3ETappable` are the reference patterns.
- **Material 3 Expressive spec**: Components align with the M3E design spec — expressive shapes, state layers, neighbor effects where appropriate (ask for confirmation when adding squish or similar behavior that may affect layout).
- **Barrel exports**: Each component exposes via its entry `m3e_*.dart`; public symbols reach consumers through [`material_3_expressive.dart`](lib/material_3_expressive.dart).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [paadevelopments/material_3_expressive](https://github.com/paadevelopments/material_3_expressive) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
