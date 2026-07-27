---
trigger: always_on
description: Catalog of every React component under `components/`. One entry per `.tsx` file. Subdirectories (`examples/`, `modals/`, `svg/`, `page/`, `detectors/`) are excluded — those compose this library, they are not the library itself.
---

# AGENTS.md — components

Catalog of every React component under `components/`. One entry per `.tsx` file. Subdirectories (`examples/`, `modals/`, `svg/`, `page/`, `detectors/`) are excluded — those compose this library, they are not the library itself.

Tests under `components/__tests__/` enforce that this catalog stays in sync with the source. Adding a component without documenting it here fails CI.

## How to read each entry

- **Path** — where the source file lives.
- **Purpose** — one sentence describing what the component does.
- **Props** — copied from the source `interface` or `type Props` block. Kept in sync by `props_sync.test.mjs`.
- **Theming tokens** — CSS custom properties (`--theme-*`, `--ansi-*`, `--font-*`, etc.) the component uses. Kept in sync by `theming_tokens_sync.test.mjs`. If none, the field reads `(none)`.
- **CLI primitive** — the equivalent in the CLI framework (`scripts/cli/lib/*`). If none exists, the field reads `(React-only)`.
- **Used by** — where the component appears in the kitchen sink or examples. Kept in sync by `component_usage_sync.test.mjs`.

This catalog tells you **what** each component is. The four `skills/port-sacred-terminal-ui-to-*/SKILL.md` files tell you **how** to port one.

## Raw component source

Every `components/*.tsx` file is served at `https://sacred.computer/llm/components/<Name>.tsx.txt`. Fetch the source over HTTP without cloning the repo.

---

## Accordion

- **Path:** `components/Accordion.tsx`
- **Purpose:** Click-to-toggle collapsible section with a title row and a children body.
- **Props:**
  ```ts
  interface AccordionProps {
    defaultValue?: boolean;
    title: string;
    children?: React.ReactNode;
  }
  ```
- **Theming tokens:** `--theme-focused-foreground`
- **CLI primitive:** (React-only) The CLI framework renders flat pages — there is no folding section concept.
- **Used by:** `<Accordion defaultValue={true} title="ACTION BAR">` in the kitchen sink (`app/page.tsx`).

## ASCIICanvas

- **Path:** `components/ASCIICanvas.tsx`
- **Purpose:** Animated ASCII art rendered in a `<pre>` element using per-cell `<span>` elements with DOM diffing.
- **Props:**
  ```ts
  { rows?: number }
  ```
- **Theming tokens:** `--font-family-mono`, `--font-size`, `--theme-line-height-base`
- **CLI primitive:** (React-only) The CLI framework is static — animation belongs on the React side.
- **Used by:** `<ASCIICanvas rows={20} />` in the "ASCII CANVAS" accordion in `app/page.tsx`.

## ActionBar

- **Path:** `components/ActionBar.tsx`
- **Purpose:** Horizontal toolbar of action items, each with optional hotkey and nested dropdown menu.
- **Props:**
  ```ts
  interface ActionBarProps {
    items: ActionBarItem[];
  }
  ```
- **Theming tokens:** `--theme-background`, `--theme-border`
- **CLI primitive:** `buttonRow` plus repeated `button(hotkey, label)` calls. The CLI version is non-nested; nested dropdowns are React-only.
- **Used by:** `<ActionBar items={[ ... ]} />` inside the "ACTION BAR" accordion in `app/page.tsx`.

## ActionButton

- **Path:** `components/ActionButton.tsx`
- **Purpose:** Hotkey + label button pair, the React peer of the CLI `button` primitive.
- **Props:**
  ```ts
  interface ActionButtonProps {
    onClick?: () => void;
    hotkey?: any;
    children?: React.ReactNode;
    style?: any;
    rootStyle?: any;
    isSelected?: boolean;
  }
  ```
- **Theming tokens:** `--theme-button-background`, `--theme-button-foreground`, `--theme-focused-foreground`, `--theme-text`, `--font-family-mono`, `--font-size`
- **CLI primitive:** `button(hotkey, label)` in `scripts/cli/lib/button.ts` (`button(hotkey, label)` in the Python mirror). Pair with `buttonRow(...)` to get the same left/right layout.
- **Used by:** `<ActionButton hotkey="ESC">EXIT</ActionButton>` in `components/examples/CLITemplate.tsx`, `components/examples/InvoiceTemplate.tsx`, `components/examples/ResultsList.tsx`, and the "ACTION BUTTONS" accordion in `app/page.tsx`. Every CLI port surface uses `ActionButton` (not `Button`) so it stays in lockstep with Simulacrum's `button(hotkey, label)` primitive.

## ActionListItem

- **Path:** `components/ActionListItem.tsx`
- **Purpose:** Menu row that renders as either an anchor or a button with a leading icon glyph.
- **Props:**
  ```ts
  interface ActionListItemProps {
    style?: React.CSSProperties;
    icon?: React.ReactNode;
    children?: React.ReactNode;
    href?: string;
    target?: string;
    onClick?: React.MouseEventHandler<HTMLDivElement | HTMLAnchorElement>;
    role?: string;
  }
  ```
- **Theming tokens:** `--theme-button-background`, `--theme-button-foreground`, `--theme-focused-foreground`, `--theme-text`, `--theme-line-height-base`, `--font-size`
- **CLI primitive:** `cardRow(formatRow([icon, label], colSpec, innerW), innerW)`. The CLI has no anchor concept — interactive items are wired through `createApp({ interactive, onKey })`.
- **Used by:** `<ActionListItem icon={'⭢'} href="https://internet.dev" target="_blank">` inside the navigation example in `app/page.tsx`.

## AlertBanner

- **Path:** `components/AlertBanner.tsx`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [internet-development/www-sacred](https://github.com/internet-development/www-sacred) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
