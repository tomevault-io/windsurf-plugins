---
trigger: always_on
description: Humanized browser automation for AI agents, QA tests, and demos. Playwright-first.
---

# HumanJS

Humanized browser automation for AI agents, QA tests, and demos. Playwright-first.

> This file is the brief loaded into every Claude session in this repo. Keep it lean.
> Architectural reasoning lives in [`docs/DESIGN.md`](./docs/DESIGN.md) (public). Strategic and personal notes live in `docs/PROJECT_BRIEF.md` (local-only, gitignored).

## Project status

Greenfield — no code yet. We're scaffolding the v1 monorepo from scratch.

## Positioning (immutable)

**HumanJS humanizes browser automation for AI agents (Browser Use, Stagehand, Playwright MCP), QA tests where real-pace timing exposes bugs, and demo/tutorial recordings.**

When writing copy, docs, examples, or comments: never frame the project around scraping, captcha bypass, or "undetectable" automation. The audience is AI agent builders, QA engineers, and demo/tutorial creators.

## Non-goals — refuse to implement

If asked, decline and link to this section.

- Captcha solvers / captcha bypass
- Fingerprint masking or browser fingerprint spoofing
- Proxy rotation or IP management
- TLS / network-level stealth
- "Undetectable" claims anywhere in any artifact
- Time-of-day variation (gimmick, no real value)
- AI-generated mouse trajectories (worse than Bezier, vastly more expensive)

## Naming

- **Brand**: HumanJS
- **Primary npm scope**: `@humanjs/*`
- **Fallback scope**: `@totigm/humanjs` (used only if `@humanjs` org isn't claimable on first publish)
- **Domain**: humanjs.dev (planned)

Verified available on npm at project start: `@humanjs/core`, `@humanjs/playwright`, `@humanjs/recorder`, `@humanjs/browser-use`, `@humanjs/stagehand`, `@humanjs/mcp`, `@humanjs/generator`, `@humanjs/skill`, `@humanjs/recipes`. Unscoped `humanjs` is owned by a dormant 2022 package — irrelevant scope, doesn't block us.

## Architecture

- **Monorepo** (pnpm workspaces, optionally Turborepo). Each package independently versioned.
- **Playwright-first.** Puppeteer adapter is roadmap, not v1.
- **Plugin system from day one.** Even if v1 ships no plugins, the architecture supports extending personalities, primitives, and behaviors via plugins. Non-negotiable architectural decision.
- **TypeScript strict.** ESM-first with CJS dual-publish where needed.
- **MIT licensed.**

## Packages

| Package | Purpose | Tier |
|---|---|---|
| `@humanjs/core` | Personality system, timing math, types, plugin contract | v1 |
| `@humanjs/playwright` | Playwright adapter — the main public API | v1 |
| `@humanjs/recorder` | Session recording → mp4 / Playwright code / JSON | v1 |
| `@humanjs/browser-use` | Browser Use integration adapter | v1 |
| `@humanjs/stagehand` | Stagehand integration adapter | v1 |
| `@humanjs/mcp` | MCP server for runtime AI agents | v1 |
| `@humanjs/skill` | Anthropic / Cursor / Cline skill for AI coding agents | v1 |
| `@humanjs/generator` | `npx @humanjs/generator <url>` — visual recorder UI | v2 |
| `@humanjs/recipes` | Pre-built common flows (login, checkout, etc.) | v2 |
| `@humanjs/puppeteer` | Puppeteer adapter | v3 |

## Public API shape — keep examples consistent with this

```ts
import { createHuman, blend } from '@humanjs/playwright';

const human = await createHuman(page, {
  personality: 'careful',  // careful | fast | distracted | precise | custom
  seed: 'optional-seed',   // deterministic when set
  speed: 'human',          // human | fast | instant
  plugins: [],             // optional plugins
});

await human.goto(url);
await human.click(selector);              // hover, micro-move, click
await human.rightClick(selector);         // context-menu click
await human.hover(selector);              // hover without clicking
await human.move(target);                 // selector | Locator | Point — positional, no settle dwell
await human.drag(from, to);               // each endpoint: selector | Locator | Point
await human.type(selector, value);        // click, then realistic typing rhythm
await human.paste(selector, value);       // Cmd-V style (no per-char timing)
await human.read(text);                   // dwell based on word count
await human.scroll('natural');
await human.press('Mod+S');               // chord — 'Mod' auto-maps: Meta on Mac, Control elsewhere
await human.press('Tab');                  // bare key — single keys work too

const rec = await human.record(async () => { /* actions */ });
await rec.toVideo('out.mp4');                // shipped
await rec.toGif('out.gif');                   // shipped
await rec.toTimeline('session.json');         // shipped
// await rec.toPlaywright('test.spec.ts');    // v0.2 — Playwright code export

await sleep(800);                             // shared helper, re-exported from @humanjs/core
```

Selector strategy: prefer accessible names + roles. Default to `getByRole`, `getByLabel`, `getByText` before falling back to CSS / XPath. This is also a humanization signal — real users navigate by what they see.

## Personalities

Layered API:

```ts
// 1. Preset
{ personality: 'careful' }

// 2. Preset + overrides
{ personality: { extends: 'careful', typing: { typoProbability: 0.1 } } }

// 3. Fully custom
{ personality: { speed, mouse, typing, reading, dwell } }

// 4. Composition
{ personality: blend('careful', 'distracted', 0.3) }

// 5. Community-published

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [totigm/humanjs](https://github.com/totigm/humanjs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-26 -->
