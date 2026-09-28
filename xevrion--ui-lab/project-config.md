---
trigger: always_on
description: <!-- BEGIN:nextjs-agent-rules -->
---

<!-- BEGIN:nextjs-agent-rules -->

# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` (resolved from this file's directory; in monorepos the `next` package may not be visible from the repo root) before writing any code. Heed deprecation notices.

This block is written and re-added by `next dev` — verify at `node_modules/next/dist/server/lib/generate-agent-files.js`. Removing it from a diff only re-creates the uncommitted change; committing it with your work keeps the tree clean.

<!-- END:nextjs-agent-rules -->

# ui lab: how to work here

ui lab (lab.xevrion.dev) is Yash's (xevrion) personal lab of small interaction experiments: things he made because he liked how they felt. It is **not a UI library** and must never be pitched as one. Every piece is a live demo with its source on GitHub. The bar is high: each component should be something a designer would screenshot and share, while staying simple, fast and accessible.

Stack: Next.js 16 App Router (read `node_modules/next/dist/docs/` before using a Next API), React 19, Tailwind CSS 4, Motion (`motion/react`), Bun. Deployed on Vercel; a push to `main` is a production deploy. `simple-icons` is available for brand logos (tree-shaken; import named icons only).

## Making a component: the workflow

1. **Understand the brief.** When Yash describes a component, follow his spec closely; ask only if something is genuinely ambiguous. If he gives a reference image, keep the idea but render it in this lab's style (monochrome tokens, restrained, one accent at most).
2. **Scaffold:** `bun run new <slug> <category>`. It creates `src/lab/components/<slug>.tsx` and registers it in three files: `src/lab/registry.ts` (metadata only, never import components there), `src/lab/demos.tsx` (code-split loader for its own page) and `src/lab/previews.ts` (static bundle for the index). Never remove the `// new-component:` markers. Categories: `buttons`, `inputs`, `navigation`, `feedback`, `data`, `cards`, `objects`, `playground`, `text` (text effects are ranked last on the index).
3. **Flag it new:** add `isNew: true,` under its `slug:` line in the registry. New entries lead the index and get a quiet "New" tag (a static pill with a red dot; the old self-drawing circle was too loud in a grid), and the newest one is linked from the hero's "New" pill. When a new *batch* lands, clear the previous batch's flags; single additions can stay alongside the latest batch.
4. **Build it** (rules below). Export the reusable component by name with sensible props, and `export default function <Name>Demo()` with believable, real content.
5. **Verify visually** (see "Seeing it"): light and dark, mid-animation frames, 375px width, console clean, reduced motion.
6. **Register:** `bun scripts/register.ts '[{"slug":"x","description":"...","keywords":"..."}]'`. Description: one short present-tense line about what it *does* ("Leans toward your cursor before you even reach it."). Keywords: 4 to 8 lowercase search words. Add `"anchor":"top"` if the demo grows downward.
7. **Fit its card:** measure the demo's rendered size at 1440px, then `bun scripts/set-scale.ts '{"slug":[w,h,max?]}'` (fits into 316x200; `max` like 1.6 lets tiny controls grow). Very tall demos can use `previewCrop: true`. Then look at the card on the index, at rest and hovered.
8. **Hover show:** add a short hover demonstration for the index card (see "Card previews"). If the component has nothing meaningful to show, add nothing.
9. `bunx eslint <file>`, `bunx tsc --noEmit`, `bun run build`. Commit only when asked (single-line message, no Co-Authored-By trailer).

Helper scripts live in `scripts/` (not the scratchpad, which gets wiped): `new-component.ts`, `register.ts`, `set-scale.ts`, `set-description.ts '{"slug":"text"}'`, `fetch-contributions.ts` (runs before every build).

## Design rules (from Yash's feedback; each one was a real complaint)

- **Simple, unique, not generic.** One signature idea per component, drawn from what the user is doing at that moment, executed perfectly. Not a shadcn/Radix/Aceternity clone, and not over-engineered: small footprint, calm at rest, alive when touched. If a component looks complex or crowded, it's wrong.
- **Real-object quality when it's an object.** Physical metaphors (dials, receipts, lamps, tape measures) must look like beautifully made objects with believable materials, not clip art or grey placeholders.
- **The thing you click never moves** out from under the cursor. Content grows downward from a fixed top (then set `anchor: "top"`); never auto-collapse siblings; never scroll on toggle.
- **Nothing appears out of thin air.** New controls unfold from the element that caused them (e.g. call controls slide out from behind the End button); the pressed button morphs into its next role rather than being replaced.
- **No muddy crossfades between hues.** Half-transparent red over green reads brown; over grey reads pink. Swap colours with a clip-path wipe or keep them on separate layers.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xevrion/ui-lab](https://github.com/xevrion/ui-lab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
