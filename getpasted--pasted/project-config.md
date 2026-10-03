---
trigger: always_on
description: Pasted aims to make its meaningful clipboard-management capabilities available through both the graphical app and the CLI.
---

# Pasted contributor guidance

## GUI and CLI parity

Pasted aims to make its meaningful clipboard-management capabilities available through both the graphical app and the CLI.

- When adding or changing a user-facing capability, consider its GUI and CLI surfaces together.
- Prefer shared Rust domain services and data contracts so the GUI and CLI execute the same behavior instead of maintaining parallel implementations.
- Keep terminology, defaults, validation, safety limits, ordering, and error semantics consistent across both surfaces.
- CLI commands should support stable, scriptable output. Prefer a consistent structured-output option such as `--json` when results contain multiple fields.
- GUI-only presentation behavior does not require a CLI equivalent. Features that depend on a running app, operating-system integration, or interactive state may use an explicit live-app command path or remain deferred.
- If a meaningful capability ships on only one surface, document the intentional exception or leave a clearly scoped follow-up before treating the work as complete.
- Add parity-focused tests where practical, especially around shared mutations and structured output contracts.

## Localization and language parity

- Treat every locale registered in `src/locales/manifest.json` as a fully supported product surface. Any change to product-authored user-facing copy must update the canonical English catalog and every shipped locale in the same change; do not rely on the English runtime fallback to ship an incomplete translation.
- Put GUI copy, native menu copy, validation, notifications, accessibility labels, alt text, user-facing errors, and formatted count/date/list text behind stable localization keys. Keep intentional technical literals, stable identifiers, user data, commands, flags, paths, and external proper names untranslated as documented in `docs/LOCALIZATION.md`.
- Reuse an existing semantic message when the meaning and grammatical role are genuinely identical. Use separate keys when context, tone, plurality, capitalization, or future translation may differ; do not assemble translated sentences from independently translated fragments.
- Preserve interpolation placeholder names and plural-message shapes across every catalog. Use locale-appropriate plural categories and shared internationalized formatters rather than concatenating English punctuation or word order around translated values.
- Machine or local-model translations are drafts. Run the locale-specific editorial review scripts after draft generation, review new strings in their interface context, and preserve established product terminology—especially for destructive actions, privacy, security, backup and restore, platform instructions, Clips, Bins, and named destinations.
- Keep React, Rust-native menus, shipped registry metadata, and the locale manifest synchronized. When adding or renaming a localization key, update every consumer and catalog atomically and remove stale keys or audit exceptions rather than leaving hidden compatibility debt.
- Before treating user-facing copy work as complete, run `npm run test:i18n` and `npm run test:copy`; run `npm run build` and relevant Rust tests when runtime, native-menu, registry, formatter, or locale-selection behavior changes. Never weaken completeness checks or increase hardcoded-copy debt to make a change pass.
- Use logical inline-start and inline-end layout utilities and CSS properties for direction-sensitive spacing, alignment, borders, corners, and positioning. Physical left/right styling requires a documented operating-system or geometry exception in the RTL audit.
- Keep interface direction separate from user content. Clipboard text and user-defined labels use automatic direction; file paths, commands, code, and stable identifiers remain isolated or explicitly LTR as appropriate.
- Mirror navigation, disclosure, and directional-flow icons in RTL. Do not mirror semantic symbols whose meaning is direction-independent.

## Theme-safe styling

- Use Tailwind utilities for structure: layout, spacing, sizing, responsive behavior, and typography.
- Use Pasted semantic theme classes and CSS custom properties for colors, surfaces, borders, dividers, focus rings, and visual emphasis.
- A structural border utility such as `border`, `border-t`, or `border-b` never supplies its own color. Pair it on the same element with a semantic theme class or a component class that explicitly sets a semantic `border-color`; never allow it to fall back to `currentColor`. Dialog panels with borders must use `theme-panel` unless a documented semantic panel class owns the complete surface treatment.
- Do not introduce Tailwind's default palette utilities directly into user-facing UI. If a reusable semantic class does not exist, add one to the theme primitives instead of special-casing a component.
- Mechanical utilities such as `divide-y` must be paired with the corresponding semantic class, such as `theme-divide`.
- Keep the CSS architecture audit budgets ratcheting downward; never increase a debt budget to accommodate new styling.

## Architecture and audit ratchets


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [getpasted/pasted](https://github.com/getpasted/pasted) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
