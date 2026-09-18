---
trigger: always_on
description: Next.js 16 (App Router) + React 19 + Tailwind **v4** + Zustand, with a Node `ws`
---

# CLAUDE.md — Custom UNO

Next.js 16 (App Router) + React 19 + Tailwind **v4** + Zustand, with a Node `ws`
game server (`server/index.ts`). This file exists to make **mobile** changes fast
and safe. The desktop game is finished and polished — treat it as frozen.

Read this before touching any layout/CSS.

---

## 1. Breakpoint system (ONE definition)

There is **no `tailwind.config.*`** — Tailwind v4 is configured in CSS at
`src/app/globals.css` (`@theme` block). Historically three different "phone"
numbers were in play (500 / 640 / 950px); that fragmentation is the #1 cause of
mobile changes cascading unpredictably. Use these canonical values:

| Token            | Value        | Where it lives                          | Meaning                                    |
| ---------------- | ------------ | --------------------------------------- | ------------------------------------------ |
| **phone**        | `≤ 500px`    | `src/hooks/useIsPhone.ts` (`PHONE_QUERY`) | The switch that matters: renders `MobileGameTable` instead of `GameTable`. |
| **phone-landscape** | `landscape and max-height:500px and max-width:950px` | `.rotate-prompt` @media, `globals.css` | Shows the "rotate to portrait" overlay.    |

**Rules:**
- **≤ 500px is the definition of "phone."** Any new phone-vs-desktop decision
  keys off `useIsPhone()`, not an ad-hoc media query or Tailwind prefix.
- **Do not add new numeric breakpoints.** If you think you need one, you almost
  certainly want a component-local `clamp()` or a `useIsPhone()` branch instead.
- **Known mismatch (leave unless asked):** `Lobby` and `NameGate` still use
  Tailwind's default `sm:` (= **640px**), which never fires on a real phone
  (≤500px). It's harmless today because those screens degrade gracefully. When
  reworking one of them, migrate its `sm:` usage toward the 500px phone model
  rather than adding more `sm:` utilities. Do **not** globally redefine `sm:`.
  (`RoundEnd` was rewritten in v2 and is now fully fluid — `clamp()` and
  `min()` throughout, no breakpoints at all. Use it as the model.)
- Do **not** raise `useIsPhone` to 640px — that would reclassify small tablets as
  phones and hand them the portrait-only mobile board.

---

## 2. No fixed pixel widths on containers

Containers, positioners, and layout wrappers use **relative units only**:
`%`, `rem`, `clamp()`, `vw`/`vh`, `min()/max()`. No magic-number pixel widths on
anything that holds layout.

**Explicit, deliberate exception — individual card sizes.** Cards (`CardFace`/
`CardBack`, and the `W = …` widths in `MobileGameTable.tsx`) are sized in fixed
px **on purpose**: the mobile hand is a fixed-card-size fan that turns into a
horizontal **swipe carousel** when it overflows (`MobileHand`), so cards stay
readable instead of crushing to slivers. Keep card widths fixed; keep their
*containers* relative. The rule targets containers, not the cards.

When you do touch card sizing, prefer `clamp(min, vw-expr, max)` (as `MobileHand`
already does: `Math.max(54, Math.min(72, vw * 0.19))`) over a bare constant.

---

## 3. Desktop-critical — do NOT alter visually without explicit approval

The desktop experience is done. These are off-limits for mobile work; a mobile
tweak must never change how any of them render:

- `src/components/GameTable.tsx` — the desktop table.
- `src/components/OpponentSeat.tsx` — desktop opponent seats.
- The desktop branch of `src/components/RoomClient.tsx` (the `!isPhone` path).
- Shared visual primitives in `src/app/globals.css`: `.card-shadow*`, the
  `@keyframes` (card-drop, wild-pop, uno-wiggle, seat-bob, arrow-*, draw-hint-*,
  focal-*), and the color/radius `@theme` tokens. These are used by desktop too.

**Where mobile work belongs:** `src/components/MobileGameTable.tsx` and
`src/hooks/useIsPhone.ts`. Mobile and desktop tables **duplicate game logic
verbatim** — this is intentional isolation, so a mobile CSS change cannot reach
desktop. The flip side: real *game-logic* fixes must be applied to **both**
`GameTable.tsx` and `MobileGameTable.tsx`.

**The exception — shared, on purpose:** `src/components/HeroScene.tsx` carries
*both* the desktop and phone hero compositions behind one `<HeroBackdrop />`,
and it backs the landing page **and** `RoundEnd`. It's shared because it has no
game logic — only art placement. Changing a layer there changes both screens, so
check both. Each composition is transcribed from its own design canvas
(desktop 3132×1762; phone 806×1752, from the Figma frame "iPhone 17 - 2",
node `1:153`) — keep layer coordinates in those canvas units so the code stays
diffable against the design.

If a change genuinely needs to alter a desktop-critical file, stop and get
explicit approval first.

---

## 4. Card rendering & scaling (PNG, not SVG)

Cards are **PNG raster** images (`/public/cards/*.png`), not SVG. Art is
**660×1029** — ratio ≈ **1.559**, *taller* than a flat 2:3.

- **Cards are width-driven:** pass a `width`, let `height` be `auto`
  (`CardFace`/`CardBack` in `src/components/Card.tsx`, via `ui/Card.tsx`).
- **NEVER force `aspect-ratio: 2/3` or `overflow: hidden` on a card.** It was
  tried and sheared ~6% off the bottom of every card (the ratio is 1.559, not

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Crisiswastaken/Uno-Groovy](https://github.com/Crisiswastaken/Uno-Groovy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-18 -->
