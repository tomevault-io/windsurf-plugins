---
trigger: always_on
description: This file applies to everything under `Tool/`. MachKit H5 tools run inside a
---

# MachKit H5 Tool Development Rules

This file applies to everything under `Tool/`. MachKit H5 tools run inside a
native macOS `WKWebView`; they must feel like one product with the SwiftUI shell,
not like unrelated web pages.

## Priorities

When requirements conflict, use this order:

1. Preserve correctness, user data, native capability boundaries, and accessibility.
2. Preserve the requested layout and behavior unless the task explicitly changes them.
3. Keep all tools visually consistent with MachKit and with each other.
4. Prefer shared components and semantic tokens over tool-specific styling.
5. Keep the bundled web payload and runtime work proportional to the tool.

## Architecture

- Use React, Vite, TypeScript, and Tailwind CSS as configured in this project.
- Source files use `.ts` / `.tsx`. Import specifiers keep the TypeScript ESM
  convention of `.js` (resolved to the `.ts` / `.tsx` implementation). HTML
  entry scripts and module workers point at the real `.tsx` / `.worker.ts` paths.
- Mount every tool with `mountTool()`.
- Use `ToolPage` and `ToolContent` for the common page shell and content geometry.
- Keep one `index.html` entry per tool under `tools/<tool-id>/`.
- Keep pure transformation logic separate from React, normally in a sibling module
  such as `json.ts`, and test it with Node's test runner via `tsx`
  (`npm test` / `node --import tsx --test`).
- Use `@/runtime/machkit.js` as the only boundary for native operations.
- Use relative asset paths. Bundled tools are loaded from local files with Vite
  `base: "./"`.

## Component Library

The repository-owned shadcn/ui component layer in `src/ui` is the standard for
all user-facing H5 controls. Radix Primitives provide accessible behavior,
Tailwind CSS provides styling, and CVA defines shared variants.

- Import shared controls from `@/ui/index.js`.
- Reuse `ToolPage`, `ToolContent`, `Section`, `Field`, `Input`, `Textarea`,
  `CheckboxField`, `Button`, `SelectControl`, `SegmentedControl`, `ValueField`,
  `InlineMessage`, and the other shared exports before adding a component.
- Use standard DOM/shadcn props such as `onClick`, `disabled`, `aria-invalid`,
  and the component's documented Radix state props.
- Use Phosphor icons. Do not introduce a second general icon library.
- CodeMirror, canvas, charts, image editors, and other specialized workspaces may
  use purpose-built libraries when the shared layer has no equivalent. Their
  surrounding controls must still use the shared shadcn/ui layer.
- Do not create a local imitation of a shared control with raw `<button>`,
  `<input>`, or a large Tailwind class string.
- Semantic HTML used only for structure (`main`, `section`, `header`, `div`,
  `code`, `pre`) does not need a component wrapper.
- Do not add HeroUI, MUI, Ant Design, Chakra, Mantine, or another competing
  general-purpose component library.

### Legacy components

Some existing tools still contain ad-hoc raw controls or tool-specific component
styles. Treat those implementations as legacy:

- Do not add new ad-hoc control styles when a shared component exists.
- Do not perform an unrelated repository-wide migration while fixing one tool.
- When a task explicitly migrates a tool, migrate all visible controls in that
  tool so the page does not mix two button, field, card, or feedback systems.
- Move reusable variants and theme adaptations into `src/ui`, not into a tool
  folder.

### Imports and bundle size

- Import components through `@/ui/index.js`; avoid deep imports from individual
  UI files outside `src/ui`.
- Use `components.json` and the shadcn workflow when a missing primitive should
  be added. Adapt generated code to the repository's tokens, naming, and
  Phosphor icon language before use.
- Keep one source file per shared component and export it from `src/ui/index.ts`.
- Before adding a dependency, confirm that the shared layer, Radix, the platform,
  or an existing dependency cannot already handle the requirement.
- Commit `package.json` and `package-lock.json` together.

### Product-pattern components

Shared shadcn primitives are the base layer. Repeated tool workflows belong in a
second repository-owned product-pattern layer under `src/ui`.

- Prefer shared patterns such as `ToolToolbar`, `ExampleChips`, `PropertyList`,
  `PropertyRow`, `StatusStrip`, `ResultPanel`, `SplitWorkspace`, `ToolSidebar`,
  `EditorPane`, `ActionGroup`, `RadioDot`, and `Slider` when they are available.
- If the same composition is needed by two or more tools, add or extend a shared
  pattern instead of copying markup and Tailwind classes into each tool.
- Product-pattern components own common geometry, density, state presentation,
  responsive behavior, and accessibility. Tool pages supply content and business
  behavior.
- Export shared patterns from `@/ui/index.js`. Do not deep-import them or create a
  parallel `components` folder inside a tool.
- A shared pattern must remain general enough for its named workflow. Do not add
  tool-specific business rules or native capabilities to the UI layer.

## Themes

MachKit has exactly two resolved visual themes: light and dark.

- `appearance="system"` is a preference mode, not a third theme. It must resolve

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rhevorn/machkit](https://github.com/rhevorn/machkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-13 -->
