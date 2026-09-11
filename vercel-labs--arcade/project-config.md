---
trigger: always_on
description: A terminal-rendered ASCII arcade: 3D games and an animated prism screen drawn with
---

# Arcade — agent guide

A terminal-rendered ASCII arcade: 3D games and an animated prism screen drawn with
truecolor half-blocks in the terminal. **Pure TypeScript, no GPU, no native deps.** Run
with `tsx`/Node and plain `pnpm`.

When shaping, editing, hardening, or reviewing user-facing UI, load
`.agents/skills/product-design/SKILL.md`. It applies to the website, terminal UI,
game HUDs, interaction flows, product copy, accessibility, responsive behavior,
and visible failure states. Skip it for backend-only, telemetry-only, generated,
or test-only work with no shipped UI impact.

For the same user-facing work, also read root [`design.md`](design.md). It links
Vercel's canonical design guidance and defines how Arcade applies that baseline
without losing its terminal, ASCII, cinematic, and Geist Pixel identity.

## Seeing your own output (read this before judging visuals)

The apps are full-screen, raw-mode, **infinite** TTY programs — do NOT run `pnpm dev` to
"see" them (you'll get escape codes and a hang). Instead, render a frame to a PNG and
view it. **See [docs/verifying-output.md](docs/verifying-output.md).** Quick version:

```bash
pnpm snapshot 140 50 0.7
sips -s format png .snapshots/prism.ppm --out .snapshots/prism.png -Z 1000
# then Read .snapshots/prism.png
```

(`.claude/settings.json` already allowlists these commands.)

## Structure

```
src/
  engine/     reusable software 3D renderer — knows nothing about the arcade (a library)
  tui/        reusable retained-mode UI library — flexbox layout, Surface compositing
  platform/   terminal control (alt screen, raw mode, SGR mouse) + input parsing
  rules/      UI-independent game states and legal-action authority
  harness/    reusable players, model prompting, communication, and match sessions
  game-visuals/ reusable renderer-only board-game models and drawing primitives
  prism/      the prism screen — a self-contained visual (scene + splash + curl/HTTP
              stream handler); depends only on engine/, shared by arcade + api/ + tools/
  auth/       Vercel sign-in (OAuth device flow) + AI Gateway key resolution
  voice/      realtime speech-to-speech session + mic/speaker I/O + echo cancel (built, not
              reachable in the shipped CLI; see docs/voice.md)
  telemetry/  anonymous usage + canonical game records → Arcade telemetry proxy → Tinybird (opt-out)
  arcade/     THE app: orchestrator (main.ts) + per-game/scene/shell presentation
    games/<game>/   per-game presentation (chess: scene, hud, turntable)
    match/          AI-vs-AI plumbing (driver, setup modal, model catalog)
    scenes/         ambient / idle-loop visuals (prism aside): logos, audio, wisp
    shell/          launcher + window chrome (cover flow, menu, bars, keybindings)
    tutorial/       the interactive walkthrough (chapters + guide rail; see docs/tutorial.md)
  tools/      snapshot.ts (render a frame to an image) + dev scripts
```

Import direction is one-way: `arcade/` consumes the libraries (`engine/` via the
`engine/index.ts` barrel, `tui/` via `tui/index.ts`, `auth/`, `voice/`, and `prism/` via
their `index.ts` barrels, plus `platform/`, `rules/`, `harness/`, and `game-visuals/`).
**The libraries never import app code** — keep it that way so they stay reusable (the goal
is to grow `engine/` into a 3D game engine and `tui/` into the shared UI toolkit for every
game). Inside a library, modules import each other directly, not through the barrel.
`prism/` is library-tier for the same reason: it's the deploy unit behind `api/` (the
`curl`-able stream), so it must not depend on the arcade.
The supported npm boundary is the `exports` map in `package.json`; shipped CLI source that
is not exported remains package implementation. See `docs/architecture/package-boundaries.md`.

## Commands

- `pnpm dev` — run the arcade (prism screen → chess / logos)
- `pnpm snapshot [cols] [rows] [t]` — render a frame to `.snapshots/prism.ppm` (`pnpm snapshot help` lists all subcommands)
- `pnpm snapshot:png …` — same, then convert the `.ppm` to a `.png` in one step
- `pnpm type-check` — `tsc --noEmit`
- `pnpm test` — unit tests via `node:test` under `tsx` (auto-discovers `src/**/*.test.ts`; no extra deps)
- `pnpm islanders:check capture` then `pnpm islanders:check` — fingerprint 24 Islanders views before a refactor
  and compare after, to prove a move of mesh or scene code doesn't change what's drawn. A pass means
  the `.ppm` snapshots are byte-identical too. The baseline is local (gitignored), not a committed
  golden file, so intended visual changes don't fight it — just re-`capture`.

The published CLI also answers `arcade --version` / `--help`, which print and exit before
any auth, network, or alt-screen work so they stay fast and scriptable.

## Issue tracking (Linear)

Work is tracked in Linear — team **AI Gateway**, project **Arcade**; issues are `AIG-###`
(e.g. `AIG-77`), poker work under the **Poker Demo Day** milestone. Via the Linear MCP,
**skip `list_teams`** (it returns the whole org) — go straight to the project: `list_issues`
with `project: "Arcade"` (add `assignee: "me"`), or `get_issue AIG-###`. Branches follow
`brianzhang/aig-###-<slug>` (Linear's suggested name). When you finish a piece of work, comment

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vercel-labs/arcade](https://github.com/vercel-labs/arcade) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-10 -->
