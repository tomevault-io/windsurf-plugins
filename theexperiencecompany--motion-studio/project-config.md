---
trigger: always_on
description: Never mention Claude, Claude Code, or any AI assistant in commit messages, PR titles, PR descriptions, or PR comments. No `Co-Authored-By: Claude …` trailers, no "🤖 Generated with Claude Code" footers, no references to AI tooling anywhere in version control history. Write commits and PRs as if a human wrote them.
---

# Project Guidelines

## Commits & PRs

Never mention Claude, Claude Code, or any AI assistant in commit messages, PR titles, PR descriptions, or PR comments. No `Co-Authored-By: Claude …` trailers, no "🤖 Generated with Claude Code" footers, no references to AI tooling anywhere in version control history. Write commits and PRs as if a human wrote them.

## Icons

Always use **HugeIcons** for all icons. Never use `lucide-react`, inline SVGs, or any other icon library.

```tsx
import { HugeiconsIcon } from "@hugeicons/react"
import { SomeIcon } from "@hugeicons/core-free-icons"

<HugeiconsIcon icon={SomeIcon} size={16} />
```

## UI Components & Styling

- Always use **shadcn/ui** components with their default styling (`Button`, `Input`, `Select`, `Accordion`, etc.)
- **Never hand-write UI components.** Always add new components via the shadcn CLI:
  ```bash
  cd packages/ui && bunx shadcn@latest add <component>
  ```
- Never use raw Radix UI primitives directly — always wrap them in shadcn-style components from `@workspace/ui/components/*`
- Never use raw HTML `<input>`, `<button>`, `<select>`, etc. with custom styling unless explicitly requested
- Always use **Tailwind** utility classes for layout and spacing — no inline styles unless required by a third-party API (e.g. Remotion player dimensions)
- The `Accordion` component in `@workspace/ui/components/accordion` uses minimal sidebar-friendly defaults (no border/rounded-2xl on root, no heavy padding on trigger). Use `className` overrides to adapt for card-style contexts if needed.

## Type Checking

Always run `bun run tsc --noEmit` (or `bun run --cwd <package> tsc --noEmit`) after completing any changes and fix all new TypeScript errors before reporting done.

## Universal Clip Style — every composition exposes the same 4 controls

Every clip in the Studio gets a "Style" section in the Inspector with **four
universal controls** that work on any non-locked composition:

- Background color
- Text color
- Font (Google Fonts picker)
- Accent color

The user expects these to *always* be there when they click a clip. Don't
hide them. Don't bake them into per-component fields. They're handled by the
shared infrastructure, not by individual `meta.ts` files.

### Architecture (don't break this)

- **`apps/remotion/src/clip-style.ts`** — declares `ClipStyle` (the four
  optional fields) and `resolveClipStyle(override, defaults)` helper. Empty
  strings count as "no override".
- **`apps/remotion/src/project.ts`** — `Clip.style?: ClipStyle` lives on
  every clip. The Studio reducer's `UPDATE_CLIP_STYLE` / `RESET_CLIP_STYLE`
  actions write here.
- **`apps/remotion/src/compositions/Project/Project.tsx`** — forwards
  `clip.style` to the rendered component as a `clipStyle` prop, **only for
  non-locked compositions**. Locked compositions never receive the prop.

### How to wire a new composition

Every new composition opts into universal styling with three lines:

```tsx
import { type ClipStyle, resolveClipStyle } from "../../clip-style";

export type FooProps = {
  // ...composition-specific props (text, value, theme, etc.)
  clipStyle?: ClipStyle;
};

export const Foo: React.FC<FooProps> = ({ /* ... */, clipStyle }) => {
  const s = resolveClipStyle(clipStyle, {
    background: "#f7f7f9",     // your composition's natural defaults
    color: "#0f1014",
    fontFamily: "-apple-system, BlinkMacSystemFont, sans-serif",
    accent: "#6366f1",
  });

  return (
    <AbsoluteFill style={{ background: s.background, color: s.color, fontFamily: s.fontFamily }}>
      {/* use s.accent for highlights / icons / springs */}
    </AbsoluteFill>
  );
};
```

**Critical rules:**

1. Use the prop name `clipStyle` (NOT `style` — collides with React's HTML
   style prop and React will pass it through to DOM nodes by accident).
2. Call `resolveClipStyle()` at the top of the component and reference the
   returned `s.background / s.color / s.fontFamily / s.accent` everywhere
   you'd previously have used hardcoded values.
3. Do **not** add per-clip `accentColor` / `backgroundColor` / `fontFamily`
   fields to the composition's `meta.ts` `fields` array. The universal Style
   section handles all four. Per-component fields are only for
   composition-specific content (text, value, layout, etc.).
4. The first argument to `resolveClipStyle` is `clipStyle` (the override);
   the second is the composition's natural defaults. Don't swap them.

### Brand-locked compositions (impersonators)

Compositions that mimic real apps **opt out** of universal styling so they
keep their authentic look. Mark them with `brandMode: "locked"` in `meta.ts`:

```ts
export const tweetCardInfo: CompositionInfo<TweetCardProps> = {
  // ...
  brandMode: "locked",
};
```

A locked composition:

- Does **not** declare a `clipStyle?: ClipStyle` prop.
- Does **not** import `clip-style.ts`.
- Hardcodes its authentic colors/fonts (Twitter blue, WhatsApp green, etc.).

The Studio's Inspector hides the Style section for locked compositions, and

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [theexperiencecompany/motion-studio](https://github.com/theexperiencecompany/motion-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-22 -->
