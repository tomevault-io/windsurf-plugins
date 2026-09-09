---
trigger: always_on
description: Guidance for AI coding agents (Codex, Cursor, GitHub Copilot, Claude Code, Windsurf, Aider, Zed, and others) working in this repository or consuming this package in another project. Humans: this is a fast, accurate map. The deeper docs are linked at the bottom.
---

# AGENTS.md

Guidance for AI coding agents (Codex, Cursor, GitHub Copilot, Claude Code, Windsurf, Aider, Zed, and others) working in this repository or consuming this package in another project. Humans: this is a fast, accurate map. The deeper docs are linked at the bottom.

## What this project is

A drop-in design system for **Unity 6 UI Toolkit** (UIDocument and PanelRenderer, UXML and USS). It ships design tokens, 42 components, 120 SVG icons, a Google Fonts typography system, a one-class mobile flip, and a small auto-attaching C# runtime. Everything is themed dark and editable from a single stylesheet. Package id: `com.sinanata.designsystem`. License: MIT. The same components render on flat screens and, on Unity 6000.5+, in world space. Both `UIDocument` and `PanelRenderer` can host flat or world-space UI; the showcase uses `PanelRenderer` for its world-space gallery, and Unity now lists `UIDocument` under UI Toolkit > Legacy while keeping it fully supported (not `[Obsolete]`, still world-space capable). Prefer `PanelRenderer` for new work.

## Golden rules (do not violate)

1. **Style with tokens and classes, never hardcoded values.** Every color, radius, spacing, and motion value comes from a `var(--...)` token in `DesignTokens.uss`. Do not write raw hex, px, or ms in component rules. If a value you need has no token, add the token first, then reference it.
2. **Never put `var(...)` in an inline UXML `style="..."` attribute.** Unity 6's clone-time `StyleVariableResolver` throws and the whole `VisualTreeAsset` fails to clone ("The UXML file set for the UIDocument could not be cloned"). Author a USS class and add the class in UXML instead. `var(...)` is fine inside `.uss` files.
3. **Class naming is BEM with a `ds-` prefix.** Block `.ds-btn`, element `.ds-btn__icon` (double underscore), modifier `.ds-btn--primary` (double hyphen), state `.is-active` / `.is-open` / `.is-spinning` (prefixed `is-`). Do not invent a new prefix or fork an existing component under a new name.
4. **The showcase is the test suite.** Every component, state, and variant must appear in `Assets/Showcase/Resources/DesignSystemShowcase.uxml`. A rule change that does not update the showcase is incomplete.
5. **`.meta` files are tracked on purpose.** Do not add them to `.gitignore`. They carry `svgType: 3` for icons and the asmdef import settings consumers rely on.
6. **A font with no fallback chain is a bug, not a default.** Unity silently serves missing glyphs from an OS font, so Arabic renders as Arial and Japanese as Microsoft YaHei *in the Editor* and as empty boxes in a WebGL build, with no warning in either. Never judge multilingual text by what the Editor shows. `DsFonts.Coverage` resolves through the explicit chain only; `Design System > Showcase > Verify Fonts` fails loudly on a gap.
7. **A fallback chain is still not enough for CJK.** Chinese, Japanese and Korean share codepoints they *draw differently*, and a chain resolves per codepoint, not per language — so the first CJK font in it wins all the shared Han and Chinese comes out in Japanese letterforms. Nothing is missing, so coverage passes and the verifier goes green. Name the face with `DsFonts.ApplyFace` when you know the language. See `docs/FONTS.md`.
8. **Every material family's solid branch starts with `dsfx_ownGeometry(f)`.** Unity 6000.5 batches *descendant* quads into an ancestor's custom-material draw, so a plain solid child (a swatch, a status dot, an active-row background) arrives in your family's fragment shader; shade it and it renders as an invisible chip of your material. Everything failing the test takes `dsfx_passthrough`. This is the single easiest thing in the FX system to regress, and it fails as a rendering oddity rather than an error. Related and equally load-bearing: call `dsfx_wellShade` **unconditionally**, never from inside a profile branch — from inside one it crashes FXC with no message. See `docs/MATERIALS.md`.
9. **Material markers are `ds-fx-` prefixed and are not styling classes.** They carry no USS rules; they are instructions read by C# (`DsFxSpec`). Rule 3's BEM grammar governs component classes — do not try to make markers obey it, and do not add USS rules for them.

## Use the design system in your project

1. Install one of three ways (README, "Installation"): copy `Assets/DesignSystem/` into your project, add it as a git submodule plus an OS-level link, or add the UPM git URL `https://github.com/sinanata/unity-ui-toolkit-design-system.git?path=/Assets/DesignSystem`.
2. Attach the master stylesheet to your UIDocument's UXML and put `ds-root` on the top element:
   ```xml
   <Style src="project://database/Assets/DesignSystem/Resources/UI/Styles/DesignSystem/DesignSystem.uss" />
   <ui:VisualElement class="ds-root">
     <ui:Button text="Get started" class="ds-btn ds-btn--primary" />
   </ui:VisualElement>
   ```
3. Build screens by composing `ds-*` classes. The canonical list of every class, its DOM, and its states is `docs/COMPONENTS.md`; the showcase UXML is the second source of truth.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sinanata/unity-ui-document-design-system](https://github.com/sinanata/unity-ui-document-design-system) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
