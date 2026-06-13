---
trigger: always_on
description: A Pokémon game engine played inside a Claude Code session. Zero dependencies, Node 18+. All data — stats, moves, sprites, locations, encounter tables, catch rates — pulled live from https://pokeapi.co. Nothing non-Pokémon ships with this project.
---

# GAUNTLET

A Pokémon game engine played inside a Claude Code session. Zero dependencies, Node 18+. All data — stats, moves, sprites, locations, encounter tables, catch rates — pulled live from https://pokeapi.co. Nothing non-Pokémon ships with this project.

Three surfaces on one ruleset:

- **Arcade** (`/gauntlet`) — the endless draft battler. Roguelike streak game; rules below.
- **Journey, web** (`npm start` → http://localhost:7779) — the web face on the Claude Code brain, multi-tenant. `server.js` (zero deps) gives every visitor a cookie identity and an isolated world under `campaigns/<uid>/` (own `.gauntlet` state + a `game.js` symlink). Player actions proxy to the engine; GM beats run `claude -p --bare` from the visitor's directory with the protocol injected from `gm-prompt.md` and tools scoped to `Bash(node game.js:*)` + Read — so the GM is the real Claude Code agent acting on that visitor's world with its own tool calls, streamed live to the UI (⚙ activity lines, then serif narration). Per-user session continuity via `--resume`.

  **Artifact test build** (`artifact/gauntlet-journey.html`) — the journey as a claude.ai artifact for instant playtesting with zero hosting. Artifact CSP blocks external fetches, so it ships self-contained: an embedded Kanto opening slice (16 species with real stats/moves/capture rates, sprites as data URIs, 5 locations with canon-ish encounter tables — Pallet Town through Mt. Moon) built from live PokeAPI at pack time. GM runs over the keyless artifact API with the directive protocol (constrained to pack content); saves via `window.storage`. Same engine math, same consequence loop. It's the demo unit — the hosted server below is the product. Rebuild the pack by re-running the pack-builder against PokeAPI with a different species/location set.

  **Hosting for non-Claude-Code users:** the operator's `ANTHROPIC_API_KEY` powers all GM beats (bare mode skips OAuth by design). Cost controls built in: `GAUNTLET_BEATS_PER_HOUR` per visitor (default 30) and `GAUNTLET_MAX_CONCURRENT` global GM beats (default 2); over-limit visitors keep full engine play, narrator-free. Deploy with the included Dockerfile (`docker build -t gauntlet . && docker run -p 7779:7779 -e ANTHROPIC_API_KEY=sk-ant-… -v gauntlet-worlds:/app/campaigns gauntlet`) or any box with Node 18+ and `npm i -g @anthropic-ai/claude-code`. The cookie is the only identity — put real auth in front of it before opening to strangers. Note: Agent SDK / `claude -p` usage on subscription plans draws from a separate monthly credit starting June 15, 2026; hosted deployments should run on an API key. Local solo play is the same server with one cookie.
- **Journey, CLI** (`/journey`) — a narrated campaign. Claude is the Game Master (protocol in `.claude/skills/pokemon-gm/SKILL.md`); the engine adjudicates every battle, catch, and purchase. Trainer card, party of 6, box, money, balls, potions. Wild encounters come from PokeAPI's real location data — `scout` on viridian-forest returns the actual Viridian Forest table with canon level ranges. Trainer and gym battles are staged by the GM (`trainer Brock geodude:12,onix:14`). Catching uses real species capture rates with the Gen-style formula. Loss is a blackout (party healed, half money), never permadeath. Story state — NPCs, plots, badges, facts — persists via `memory` namespaces and `journal`. Staged `consequence` events give the world initiative: the engine tracks location, battle-count, and session triggers, surfaces due events on the trainer card, and the GM fires them into the story. All on disk under `.gauntlet/`.

The GM loop, both modes: **CONTEXT → DECIDE → EXECUTE (engine) → PERSIST → NARRATE.** The engine owns all numbers; Claude never rolls or invents mechanics.

Every player decision is one CLI command. State persists in `.gauntlet/state.json`, so each command is stateless-in, full-frame-out: it resolves the action and prints the complete rendered screen.

## Rendering: two channels

When stdout is captured (a Claude Code tool call), the game emits **two frames per command**:

- **stdout** — a compact, escape-free frame (~3–5 KB). Sprites are luminance-shaded Unicode pixel art (`░▒▓█`). This is what the agent reads and relays; it renders cleanly in the transcript.
- **/dev/tty** — the full 24-bit ANSI color frame, painted directly onto the user's terminal, bypassing tool-output capture. Half-block sprites in true color.

When stdout is a TTY (the user runs the game directly), only the color frame is printed. `GAUNTLET_TTY=0` disables the /dev/tty channel; `NO_COLOR=1` forces plain everywhere. Never pipe the color frame into the transcript — raw ANSI does not render there and bloats context.

## How Claude plays referee

When the user wants to play (`/gauntlet`, "let's play", a move instruction):

1. Run `node game.js status` to resume, or `node game.js start` for a fresh run.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [darkly22/gauntlet](https://github.com/darkly22/gauntlet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-13 -->
