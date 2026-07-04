---
trigger: always_on
description: This file is read by Claude Code on every session. Follow everything here without being asked.
---

# CLAUDE.md — Scamp App

This file is read by Claude Code on every session. Follow everything here without being asked.

---

## Project Overview

Scamp is a local-first Electron design tool. Users draw rectangles on a canvas and the app auto-saves real TSX + CSS Module files. Bidirectional sync means external edits (agent or manual) reload the canvas. The stack is Electron + electron-vite + React + TypeScript + Zustand.

See `prd-scamp-poc.md` for full product context.

---

## Non-Negotiable Rules

- **Never write `any`** in TypeScript — use proper types or `unknown` with a type guard
- **Never skip tests** for anything in `src/renderer/lib/` — these are pure functions and must be fully tested
- **Never use `console.log` for debugging** — use the logger utility or remove before committing
- **Never hardcode IPC channel name strings** — always use constants from `src/shared/ipcChannels.ts`
- **Never read from disk in the renderer** — all file operations go through IPC
- **Never assume a case-sensitive filesystem.** Linux CI is case-sensitive; macOS APFS (the default for dev machines) is case-insensitive, so `fs.access("components/button/button.tsx")` resolves to the `Button` folder. Use directory listings + string comparison when a check depends on exact casing, and design code/tests to behave the same on both platforms.
- **In Playwright tests, use `ControlOrMeta+` not `Control+`** for keyboard shortcuts that should be cross-platform. CodeMirror's `Mod-*` bindings (and most macOS-aware app shortcuts) map to `Cmd` on macOS and `Ctrl` on Linux. A literal `Control+End` is a no-op on macOS and the next `keyboard.type` splices into wherever `editor.click()` left the cursor — usually mid-line, producing CSS the parser then rejects.

---

## Code Standards

### General:
- Prefer writing clear code and use inline comments sparingly
- If a comment would be more than ~2 lines, OR captures multi-file architectural context, move it to a new or existing file under `docs/notes/` and reference it inline (`// see docs/notes/<slug>.md`). Inline comments stay short and only encode local WHY (incidents, browser quirks, hidden invariants).
- When you make a code change that affects an existing `docs/notes/` file, update the note in the same commit. Stale notes are worse than no notes.

### TypeScript

- Strict mode is on — no exceptions
- Prefer `type` over `interface` unless you need declaration merging
- All function arguments and return types must be explicitly typed
- No implicit returns in functions that should return a value
- Use `const` by default; `let` only when reassignment is necessary
- Avoid `!` non-null assertions — handle nulls explicitly

```ts
// ✅
const getElement = (id: string): Element | null => {
  return elements[id] ?? null;
};

// ❌
const getElement = (id: string) => {
  return elements[id]!;
};
```

### React

- Functional components only — no class components
- One component per file
- Props types defined in the same file as the component, above it
- No prop drilling more than 2 levels — use Zustand store instead
- `useEffect` dependencies must be complete and correct — no suppression comments
- Event handlers named `handle[Event]` (e.g. `handleClick`, `handleMouseDown`)

```tsx
// ✅
type Props = {
  elementId: string;
  onSelect: (id: string) => void;
};

const ElementRenderer = ({ elementId, onSelect }: Props): JSX.Element => { ... };

// ❌
const ElementRenderer = (props: any) => { ... };
```

### File and Folder Naming

- React components: `PascalCase.tsx`
- Hooks: `camelCase.ts` prefixed with `use` (e.g. `useDrawTool.ts`)
- Utilities and lib files: `camelCase.ts`
- Test files: `[filename].test.ts` in `test/` at root
- CSS Modules for app UI: `ComponentName.module.css` alongside the component

### CSS Modules

- Reference theme variables (`var(--border)`, `var(--accent)`,
  `var(--text-primary)`, …) rather than hex literals. The full token
  set is declared in `src/renderer/src/styles/theme.css`.
- Add a new token to `theme.css` when an existing one doesn't fit —
  don't reintroduce raw hex values. Tokens are semantic
  (`--bg-raised`, `--text-secondary`), not numeric scales.
- One-off colors that genuinely have no reuse (e.g. a specific brand
  illustration) can stay as literals, but flag in a comment.
- The user-facing `theme.css` inside a project is a SEPARATE file
  (design tokens for the user's exported CSS) — don't conflate it
  with the app's chrome theme.

### Imports

- Use path aliases — never relative `../../..` chains more than one level deep
- Alias `@renderer` → `src/renderer`
- Alias `@lib` → `src/renderer/lib`
- Alias `@store` → `src/renderer/store`
- Group imports: external packages first, then internal aliases, then relative — blank line between groups

```ts
// ✅
import { useEffect } from 'react';
import { css } from '@codemirror/lang-css';

import { useCanvasStore } from '@store/canvasSlice';
import { parseCode } from '@lib/parseCode';

import { SelectionOverlay } from './SelectionOverlay';
```

### Zustand

- One slice per domain — never put everything in one store file
- Slices export their own types
- Actions are defined inside the slice alongside state
- Never mutate state directly — use Immer or spread

### IPC (Electron)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [angiehemans/scamp](https://github.com/angiehemans/scamp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
