---
trigger: always_on
description: Hand this to Claude Code as the starting context for the repo. It contains every
---

# Yöte — project brief

Hand this to Claude Code as the starting context for the repo. It contains every
decision made so far, the exact design values pulled from Figma, the bugs already
found and their fixes, and the questions still open.

Rename this to `CLAUDE.md` at the repo root if you want it loaded automatically
in every session.

---

## 1. What this is

**Yöte** is an open source React component library. It does form inputs, and only
form inputs, extremely well. Styled and animated out of the box rather than
headless.

The name comes from the Finnish *syöte*, meaning input. It is pronounced "yoat"
in English. Sister project to Luotain, which is Finnish for sonar probe. The
Finnish naming is deliberate house style and should continue for future projects.

**Positioning.** Radix, Base UI and Ark own the headless accessibility primitive
space and we are not competing there. What none of them ship is craft: the error
transition, the focus ring timing, the way a digit lands in a cell. That is the
product. Beautiful defaults, small prop surface, zero dependencies.

**Reference point.** Sonner by Emil Kowalski. Not for its feature count, which is
tiny, but for its philosophy: insert it and it works, the defaults are excellent,
and the documentation site lets you touch the thing before you install it.

**Hard scope rule.** This library does inputs. Not selects, not date pickers, not
form state management, not validation logic. Validation state is accepted as a
prop. We never own validation. Write this in the README so there is something to
point at when the temptation arrives.

---

## 2. Naming and domain decisions, already made

| Thing | Decision | Notes |
| --- | --- | --- |
| npm package | `yote-ui` | Bare `yote` is taken by a dormant CLI from Fugitive Labs, last published ~5 years ago. Check `@yote/*` scope availability first; if free, `@yote/input` is preferable long term. |
| GitHub repo | `Tsavsar/yote` | Brand is Yöte everywhere that matters. The install string is not the product name. |
| Docs domain | `yote.shatermt.com` | Precedent: `sonner.emilkowal.ski`, `vaul.emilkowal.ski`. Costs nothing, ships today, and links the library back to the portfolio. |
| Avoid | `.io` | Most expensive renewal of the realistic options, plus live ccTLD uncertainty from the UK–Mauritius Chagos treaty signed May 2025. Not urgent, but no upside. |
| Optional later | `yote.dev` | Cheap, Google operated, HSTS preloaded. Grab it if available even if unused. |

**Brand colour.** `#7D52F4`. Note that `hsla(256, 88%, 64%)` resolves to `#7E52F4`,
one value off in red. The Figma variable `feature-base` is `#7D52F4` and that is
canonical. Use the hex, not the HSL.

In OKLCH: `oklch(0.578 0.229 289.7)`.

---

## 3. Repository structure

npm workspaces monorepo. The docs site imports the local package directly so the
two can never drift.

```
yote/
├── package.json                 workspaces: ["packages/*", "apps/*"]
├── CLAUDE.md                    this file
├── README.md
├── packages/
│   └── yote-ui/
│       ├── package.json
│       ├── tsconfig.json
│       ├── tsup.config.ts
│       └── src/
│           ├── index.ts         barrel export
│           ├── styles.css       all component CSS, inside @layer yote
│           ├── pin-input.tsx    FIRST COMPONENT
│           ├── input.tsx        scaffolded, comes second
│           └── lib/
│               ├── use-composed-ref.ts
│               └── set-native-value.ts
└── apps/
    └── site/                    Next.js App Router
        ├── app/
        │   ├── page.tsx         landing
        │   ├── docs/
        │   │   ├── layout.tsx   sidebar shell
        │   │   ├── page.tsx     getting started
        │   │   ├── pin-input/page.tsx
        │   │   ├── theming/page.tsx
        │   │   └── accessibility/page.tsx
        │   └── layout.tsx
        └── components/
            ├── predictive-arc.tsx   the WebGL background
            ├── preview.tsx          preview + code panel shell
            └── state-switcher.tsx
```

---

## 4. Build order

1. `packages/yote-ui` scaffold: package.json, tsconfig, tsup config, empty barrel
2. `styles.css` token layer
3. `pin-input.tsx` with all five states
4. `apps/site` scaffold, Next.js, imports the workspace package
5. Landing page with the shader background and a live pin input hero
6. Docs routes
7. Deploy to Vercel, point `yote.shatermt.com`
8. Only then: `input.tsx`, the text field

Do not start the text field until the pin input is finished and deployed. One
finished component beats two half-built ones.

---

## 5. Token layer

Names mirror the Figma variables one to one. All values are CSS custom properties.
Nothing in a component is hardcoded except layout structure.

```css
:root {
  --yote-bg-default: #ffffff;
  --yote-bg-surface: #f7f7f7;
  --yote-stroke-soft: rgba(0, 0, 0, 0.05);
  --yote-text-strong: #171717;
  --yote-text-sub: #5c5c5c;
  --yote-text-soft: #8a8a8a;

  --yote-feature-base: #7d52f4;
  --yote-feature-dark: #351a75;
  --yote-purple-alpha-24: rgba(125, 82, 244, 0.24);

  --yote-error-base: #fb3748;
  --yote-error-faint: #ffc0c5;

  --yote-radius-xl: 24px;
  --yote-shadow-xs: 0 2px 4px -1px rgba(0,0,0,0.02), 0 5px 13px -5px rgba(0,0,0,0.05);

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Tsavsar/yote](https://github.com/Tsavsar/yote) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-18 -->
