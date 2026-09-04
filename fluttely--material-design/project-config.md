---
trigger: always_on
description: Guidance for Claude Code when working in this repository. These rules override any
---

# CLAUDE.md

Guidance for Claude Code when working in this repository. These rules override any
default tool behavior. When a rule here conflicts with a tool's default template,
this file wins.

> **This file is tracked in git as of 1.8.0.** It used to be gitignored alongside
> `documentation/`, on the reasoning that it was personal working notes. It is not —
> it is the repository's own account of why its API decisions are what they are, and
> keeping the reasoning out of the repository put it exactly where nobody could find
> it. Edit it in a commit like any other file. It stays out of the published archive
> via `.pubignore`: consumers install a package, not a contributor's rulebook.

## What this package is

`material_design` is a **Material Design 3 design contract** for Flutter, published on
pub.dev. It is not a component library and not a loose bag of constants: it expresses
the official M3 scales (spacing, shape, type, motion, elevation, opacity, …) as Dart
**types**, so that an app consuming it adheres to the M3 spec by construction.

Three principles define every API decision:

1. **Primitive replacement.** Public APIs accept M3 extension types
   (`M3SpacingValue`, `M3CornerValue`, `M3BorderWidthValue`, `M3OpacityValue`,
   `M3IconSizeValue`, `M3BreakpointValue`, `M3ElevationDpValue`, `M3ZIndexValue`,
   `M3ToneValue`) instead of bare `double`/`int`. No scale may quietly accept a raw
   number.
2. **No token that must be unwrapped to be used.** Scalar tokens are `static const`
   values used directly (`M3Spacings.s16`), never enums with a `.value` getter —
   the unwrap costs `const`-ness. Enums remain correct for **composite tokens**
   (`M3Motion`, `M3Elevation` — you read two fields together) and **selectors**
   (`M3ScreenSize`, `M3InteractionState` — they name a situation, not a value).
3. **Deviation is deliberate and greppable.** Extension types are erased at runtime;
   a cast always defeats them. The single sanctioned escape hatch is `M3Contract.*`
   (`M3Contract.spacing(18)` etc.). Never add a second way to smuggle an off-scale
   value in, and never use `M3Contract` inside the package itself.

Any new API must state which M3 spec section it implements (link to the
https://m3.material.io/ page in its doc comment) and use only spec values.

## Scope boundary (decided, not up for re-litigation)

**This package does not ship Material Design 3 components.** Flutter's Material
library owns `FilledButton`, `NavigationBar`, `Menu`, and the rest. A second
implementation here becomes migration debt for every consumer the day Flutter changes
its own. What the package ships is the contract those components are built from —
tokens, scales, wrappers, and the per-component *measurements* (`M3ButtonHeights` and
friends in the component token layer).

The single exception: an M3 Expressive widget Flutter does not have yet
(`M3ELoadingIndicator` today). Each such widget is a stopgap, marked `@experimental`,
and is removed once Flutter ships the real one. Do **not** implement the 2025
Expressive components (button groups, split button, FAB menu, toolbars) while
flutter/flutter#168813 is in flight — re-evaluate each release instead.

Before adding any widget, ask: does Flutter already have this, or is it about to? If
either answer is yes, ship the tokens and stop.

## Repository layout

| Path | Role | Tracked? |
| :--- | :--- | :--- |
| `lib/` | The package. `material_design.dart` is a pure export barrel over `lib/src/*.dart` module barrels. | git + pub |
| `test/` | Package tests, mirroring `lib/src/` structure. | git, not pub |
| `example/` | **Single-file** consumer example (`example/lib/main.dart`). Rendered by pub.dev. Intentionally one file so a reader can copy-paste it whole. Never split it into multiple files. | git; pub keeps only `lib/` + `pubspec.yaml` |
| `demo/` | Full Flutter web app — the live, interactive showcase deployed to GitHub Pages (https://fluttely.github.io/material_design/). Every destination has two pages: the token family rendered (`showcase_pages/`) and the code that produces it (`recipe_pages/`). | git, not pub |
| `documentation/material_design/` | Obsidian vault: design notes, M3 reference docs, roadmap/plan. Bilingual: `en_US/` is the source of truth, `pt_BR/` mirrors it. | git, not pub |
| `CLAUDE.md` | This file — the reasoning behind the API decisions and the working rules. | git, not pub |
| `tool/` | The verification gate. `verify.sh` runs everything; `check_triad.dart`, `check_changelog.dart` and `check_context.dart` enforce the rules below that no compiler can. | git, not pub |
| `.claude/` | Agent-facing context: `hooks/` (format-on-write, git guard), `commands/` (`/new-scale`, `/release`, `/verify`) and `skills/` (`commit`). Tracked — it is shared project context, not personal config. Only `settings.local.json` stays local. | git, not pub |
| `.github/workflows/` | CI: `tests.yml` (runs `./tool/verify.sh` on PRs to `main`), `deploy-demo.yml` (builds `demo/` web to gh-pages). | git |

### Module architecture

Nine modules with a one-directional dependency graph:

```text
tokens ──┬─> shape ──┬─> interaction
         ├─> layout ─┤
         ├─> color ──┴─> adaptive
         └─> typography

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fluttely/material_design](https://github.com/fluttely/material_design) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-04 -->
