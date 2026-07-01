---
trigger: always_on
description: **audio/ui** is an open-source library of accessible, composable Audio UI components for React — built on top of [shadcn/ui](https://ui.shadcn.com/), designed to be copied, pasted, and owned.
---

# audio/ui

**audio/ui** is an open-source library of accessible, composable Audio UI components for React — built on top of [shadcn/ui](https://ui.shadcn.com/), designed to be copied, pasted, and owned.

- Site: https://audio-ui.xyz · Repo: https://github.com/ouestlabs/audio-ui
- Registry endpoint: `https://audio-ui.xyz/r/{name}.json`

---

## Monorepo

Turborepo + Bun workspaces. **Always use `bun`, never `npm` or `pnpm`.**

```
audio-ui/
├── apps/
│   ├── www/        # Next.js docs site (Fumadocs + MDX) → localhost:3000
│   └── sandbox/    # Dev sandbox
└── packages/
    ├── ui/         # @audio-ui/react — published npm package (headless primitives)
    ├── utils/      # Shared utilities
    └── configs/    # Shared tsconfig, etc.
```

## Key Commands

```sh
bun run dev             # Start all apps
bun run build:pkg       # Build @audio-ui/react (what gets published to npm)
bun run lint:fix        # Auto-fix with Biome
bun run changeset       # Create a changeset before versioning
```

From `apps/www/`:
```sh
bun run build:registry  # Regenerate registry JSON — run after adding any component
```

## Stack

Next.js App Router · Fumadocs + MDX · Tailwind CSS v4 · shadcn/ui + Base UI · Phosphor Icons · Biome/Ultracite

## Where Things Live

### `@audio-ui/react` — headless primitives (`packages/ui/src/`)

- `primitives/` — Fader, Knob, Transport, XYPad, ChannelStrip (unstyled, accessible)
- `hooks/interactions/` — usePointerDrag, useWheel, useKeyboardNavigation, useFocus
- `hooks/state/` — useControlledValue, useValueAsRef

This package is **published to npm**. Don't break its public API without a changeset.

### Registry (`apps/www/src/registry/default/`)

Styled components users install via `npx shadcn@latest add @audio/player`:

```
ui/audio/player.tsx # single file: provider, player controls, tracks, queue, playback-speed
ui/audio/elements/ # transport, fader, knob, xypad, channel-strip (styled wrappers)
blocks/            # Pre-assembled ready-to-use patterns (block-{component}-{variant}.tsx)
examples/          # Usage demos
lib/               # Utilities and stores (audio-store, etc.)
```

Registry config files — **update these when adding anything**:
- `registry/registry-ui.ts` · `registry/registry-components.ts`
- `registry/registry-examples.ts` · `registry/registry-blocks.ts` · `registry/registry-lib.ts`

### Docs (`apps/www/src/content/docs/`)

MDX files. Use existing custom components (`<Install>`, `<CodeTabs>`, `<Preview>`) — not raw HTML.

## Registry Rules

> [CONTRIBUTING.md](../CONTRIBUTING.md)

**After any registry change:** run `bun run build:registry` from `apps/www/`, then `bun run lint:fix`.

**Registry dependency format:**
- audio/ui items → `"@audio/{name}"`
- official shadcn/ui → plain name (e.g., `"button"`)

**Block rules** (`registry/default/blocks/`):
- Filename: `block-{component}-{variant}.tsx`
- Export: `export default function Block{Component}{Variant}()` — PascalCase of filename, no params
- `"use client"` only if the block uses hooks, state, or browser APIs
- Icons: named imports from `@phosphor-icons/react` only, no numeric `size` prop — use `className="size-4"`
- Icon-only interactive elements must have `aria-label`; decorative icons get `aria-hidden="true"`
- Static data defined outside the component function
- Semantic color tokens only — never raw Tailwind colors (`text-gray-500`, `bg-red-500`, etc.)
- Don't add `border-border` — it's the default
- React imports: named only, never the namespace (`import * as React`)

---

## Versioning & Publishing

Uses **Changesets**. Never edit `package.json` versions manually.

```sh
bun run changeset          # Describe what changed
bun run version-packages   # Bump versions
bun run release            # Build + publish
```

---

## General Behavioral Guidelines

Behavioral guidelines to reduce common LLM coding mistakes. Merge with project-specific instructions as needed.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ouestlabs/audio-ui](https://github.com/ouestlabs/audio-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
