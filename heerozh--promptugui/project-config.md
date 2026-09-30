---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

PromptUGUI is a Unity 6+ UPM package that translates compact `.ui.xml` files into runtime uGUI hierarchies. Target use case: games that ship PC widescreen and mobile portrait from one description, with fully theme-swappable skins. Supports both **procedural high-definition surfaces** (the `<Style>`/`<Theme>` primitives drive SDF fills, glass, shaped corners and decorations — no textures required); **sprite-based aesthetics** such as pixel art (`.pxl`).

The library is **content-agnostic at runtime**: it never reads the filesystem itself. Callers register a `Func<string, Awaitable<string>> SourceResolver` that maps an opaque `src` key to XML content; how the user obtains that content (Resources, Addressables, custom paths) is their concern. Built-in helpers: `UI.UseResourcesResolver(rootPath)` and (when `com.unity.addressables` ≥ 1.0 is installed) `UI.UseAddressableResolver()`.

请始终**使用中文回答** 特别是在执行 superpowers:brainstorming 或 superpowers:writing-plans 时，请依旧使用中文。

## Canonical Design Sources

`docs~/superpowers/specs/2026-05-07-promptugui-description-language-design.md` is the master spec for the description language and C# API. Per-milestone specs and plans live alongside it. Always read the master spec before changing public API or XML semantics — section numbers (e.g. "spec §7.6") are referenced throughout the codebase and PR descriptions.

The LLM-facing authoring guide is split into three skills under `.claude/skills/`. **Any functional change or addition must be reflected in the relevant skill(s) in the same PR (in english).**

- `authoring-promptugui-xml/SKILL.md` — XML markup: built-in tag catalog, common attributes, anchor / size / margin / layout groups / Image fit / mask / Canvas-scaler / Variant / Template / Import / `if=` / `<Icon>` tag / i18n markup / Color tokens / XML parse errors. Per-control & per-feature deep dives live in `authoring-promptugui-xml/reference/*.md` (loaded on demand via pointers in the main doc): `animations.md` (`<Trigger>` / `<Animation>`), `states.md` (Btn/Tab/Toggle state visuals — `*Color` / `*Modulate` / `<Show on="state-*">` / `pressedSprite` / `selectedSprite`), `controls-tabs.md`, `controls-carousel.md`, `controls-progress.md`, `icons.md` (SpriteSet discovery & icon-name resolution).
- `scripting-promptugui-csharp/SKILL.md` — C# bridge: `UI.*`, `IScreen`, `IControl`, `ControlRegistry`, `Variants`, `[UIAttr]` / `[Bind]`, `BindItems` / `BindOptions`, Resources-backed icon / .po loading, `UI.CanvasConfigurator`.
- `using-promptugui-addressables/SKILL.md` — Addressables-backed loaders for `.ui.xml`, `.po`, and icon atlases (gated by `PROMPTUGUI_HAS_ADDRESSABLES`).

Triggers requiring a SKILL update (route to the relevant file):

- New / removed / renamed XML elements (e.g. adding a `<Toggle>` builtin, retiring `<Btn>`) → XML skill
- New / removed / renamed attributes on any built-in tag, including type changes → XML skill
- Changes to anchor / size / margin / Variant / Template / Import / `if=` semantics → XML skill
- Public C# API surface changes (anything callers touch: `UI.*`, `IScreen`, `IControl`, `ControlRegistry`, `Variants`, `[UIAttr]` / `[Bind]`) → C# skill (Addressables skill if the change is `PROMPTUGUI_HAS_ADDRESSABLES`-gated)
- Changes to the `id` path / scoping rules → both XML (declaration) and C# (`Get<T>` path) skills
- New / changed parser-time errors that authors will hit → XML skill

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Heerozh/PromptUGUI](https://github.com/Heerozh/PromptUGUI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
