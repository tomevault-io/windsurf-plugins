---
trigger: always_on
description: - This repository ships the existing Node package and the Go CLI/GUI as parallel products. Do not treat the Go implementation as permission to remove or silently diverge from the Node implementation.
---

# Repository context

- This repository ships the existing Node package and the Go CLI/GUI as parallel products. Do not treat the Go implementation as permission to remove or silently diverge from the Node implementation.
- Use pnpm for every JavaScript workspace in this repository. Do not use npm. Keep the root and `go/gui/frontend` lockfiles independent.
- Use PowerShell syntax for local commands on Windows and prefer `/` in generated paths.

# Shared behavior and fixtures

- Treat `fixtures/` as the cross-implementation contract. When a fixture exposes a parser or metadata bug, update both Node and Go behavior unless the user explicitly narrows the scope.
- Keep fixture coverage capability-driven: cover distinct DOM and metadata shapes such as single-disc, multi-disc, no-cover, tabbed lyrics, and fallback artist/composer cases instead of collecting redundant pages.
- Do not automate THBWiki crawling or fixture refreshes. THBWiki fixtures are maintained from user-provided local captures because the site has strict anti-scraping behavior.
- Keep `fixtures/`, `go/`, and test data out of the published Node package. Preserve the package-boundary check in CI.

# Go architecture

- Keep reusable behavior in `go/internal`: domain types in `domain`, orchestration in `application`, source integrations in `source`, and composition in `bootstrap`.
- Keep `go/cmd/thtag` and `go/gui` thin. The CLI and GUI should call the same application services instead of reimplementing scan, planning, write, rename, rollback, image, or metadata logic.
- Preserve explicit filesystem safety in write and rename flows: preview before commit, detect post-preview changes, stage writes, and retain rollback behavior.
- Use the pinned Go and Wails versions from `go/go.mod` and `go/gui/Taskfile.yml`. Do not independently upgrade Wails or its generated bindings.
- Windows icons and version resources belong beside the corresponding `main` package as architecture-specific `.syso` files. Generate GUI resources through the existing Wails task and keep CLI and GUI branding sourced from `assets/logo.ico`.

# GUI architecture and design

- Build the desktop app with Wails v3, Vue 3, TypeScript, Vite, PrimeVue, and Pinia. Frontend source must use `.ts`/`.tsx`; do not add Vue SFC (`.vue`) files or restore `vue-tsc`.
- Keep Wails access behind the `GUIApi` boundary in `go/gui/frontend/src/api`. Components and stores must not import generated Wails services directly. Maintain matching native and fixture adapters when the API changes.
- Do not hand-edit `go/gui/frontend/bindings` or `go/gui/frontend/embed/dist`; regenerate them through the Wails/Vite tasks.
- Keep business rules in Go. The frontend owns presentation, interaction state, and explicit DTO adaptation only.
- Prefer a compact desktop-tool layout: avoid card-heavy page framing, redundant descriptions, and oversized controls. Apply visual changes consistently across tagging, batch, settings, dialogs, empty states, and loading states.
- Loading must not introduce layout shifts, duplicate cover placeholders, or transient controls that cannot be used. Derive UI state from the active operation instead of maintaining parallel flags.
- Do not add hand-written accessibility markers in GUI markup. Remove manual `aria-*`, `role`, and `tabindex` attributes, and do not override accessibility attributes generated internally by UI-library components.
- Hand-written TSX may use only native `div` elements, except that shared form-field components may use `label` to preserve native control focus behavior. Use shared components or PrimeVue components for controls and media; do not use other semantic native elements. When repeated styling previously depended on a semantic element's browser defaults, extract a shared component with explicit classes and preserve attribute passthrough.

# GUI verification

- For layout and interaction work, start the Vite frontend and use `http://127.0.0.1:9245/?fixture=1`. Fixture mode must remain offline and must not modify real music files.
- For browser-based GUI inspection, explicitly select the connected Chrome plugin/extension. Chrome control belongs to Browser Use, not Computer Use. Do not use automatic browser selection or the built-in app Browser unless the user explicitly requests it.
- If the Chrome plugin is unavailable, disabled, disconnected, or cannot control the target tab, stop browser verification and report the problem. Do not fall back to the built-in app Browser or Computer Use without explicit user approval.
- Do not launch standalone Playwright or install it for GUI verification. Browser Use's structured page inspection API is allowed because it operates through the selected Chrome connection.
- Use Computer Use only for native desktop applications, operating-system UI, cross-application workflows, or when the user explicitly requests it.
- Inspect GUI behavior with targeted DOM/state reads and scoped screenshots; avoid full-page snapshots that embed large cover images.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [the1812/Touhou-Tagger](https://github.com/the1812/Touhou-Tagger) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
