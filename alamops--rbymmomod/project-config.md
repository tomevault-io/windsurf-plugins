---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

`RBYMMOMod` (mod id `rby_mmo`) — a **mod for
[Gen1Recomp](https://github.com/bryanthaboi/gen1recomp)**, not a standalone game and not a fork
of the engine. It adds shared-overworld multiplayer: presence, nameplates, chat bubbles, scoped
chat, and trade/battle from anywhere.

Nothing here runs on its own; it needs a Gen1Recomp checkout. Everything below the "Upstream
engine" heading is the contract this mod is written against, gathered from `bryanthaboi/gen1recomp`
(branch `dev`, MIT).

### Layout

```
manifest.json        api 2, permissions:["network"], affects_link:false, experimental:true
main.lua             entry chunk: a mod:read-based resolver, then Client.install()
src/Config.lua       constants (PROTOCOL, intervals, radii, sprite list)
src/Wire.lua         message-type vocabulary + the sanitiser every inbound field passes
src/Transport.lua    hub connection, built on src/link/Net.lua's TCP framing
src/Roster.lua       who is online and where
src/Avatars.lua      remote players as runtime NPCs, driven by scriptMove
src/Chat.lua         scrollback + speech bubbles
src/Party.lua        the two-player party, and the invite that forms one
src/Overlay.lua      render.hud drawing (nameplates, bubbles)
src/Ui.lua           every registered screen
src/SessionNet.lua   a Net-shaped shim so engine link code runs over the hub
src/Sessions.lua     requests, handshake, then handoff to TradeSession / LinkBattle
src/World.lua        guarded mod.world:current()
src/Client.lua       wiring: options, hooks, events, inbound dispatch
server/hub.js        the hub (node, no deps, newline-JSON over TCP)
tests/               the mod's suite (excluded from the packed archive)
```

### Three decisions worth not re-litigating

1. **The mod ships its own hub.** The engine's relay is hard-capped at two players
   (`join_error: full`; the ENet backend disconnects a third peer), so a shared world could not
   reuse it. The hub relays only — it never simulates a battle.
2. **Trade and battle are not reimplemented.** `Protocol.TradeSession` and `LinkBattle` are
   driven over `SessionNet`, which answers the five things `LinkBattle` touches (`send`, `poll`,
   `update`, `close`, `.closed`). This is a third Net backend living in a mod.
3. **`affects_link` stays `false`.** The suite asserts the link surface is byte-identical with
   the mod installed. If that assertion ever fails, the mod started writing into a link registry
   and every player's fingerprint moved.

### Commands

The engine checkout lives at **`~/Projects/alamops/gen1recomp`** (branch `dev`),
with this folder symlinked in as `mods/rby_mmo`. Run everything from there:

```sh
python3 tools/modkit.py validate mods/rby_mmo   # loads it through the real loader
python3 tools/modkit.py lint mods/rby_mmo       # the no-ROM-bytes floor
python3 tools/modkit.py pack mods/rby_mmo       # warnings are fatal here
luajit mods/rby_mmo/tests/rby_mmo_test.lua      # the mod's own suite
luajit tests/run_modkit.lua                     # T4; auto-discovers the suite above
```

From this folder: `node server/hub.test.js` (starts the hub and drives it over real sockets).

For the real thing — two LÖVE instances, real sockets, real menus — needs a ROM
imported first (`scripts/setup.sh --rom "…"`), then:

```sh
bash mods/rby_mmo/tests/drivers/run-mmo-e2e.sh
```

**`modkit validate` passing is weaker than it looks here.** `experimental: true` means the loader
leaves the mod disabled, and a disabled mod's entry chunk never runs — so validate can go green
without executing a line of `src/`. The Lua suite is what actually exercises it: it loads the mod
through a filesystem whose `options.lua` enables it, then asserts the screens, hooks and exports
are really installed.

## Upstream engine (the thing being modded)

Gen1Recomp is a native **LÖVE2D / Lua** recreation of Gen 1 Poké Red, Blue and Yellow. The
engine is hand-written Lua; game data and graphics are decoded at runtime from a ROM the player
supplies. **The engine ships no ROM and no extracted data, and neither may this mod** — see
"Legal posture" below.

- Runtime: **LÖVE 11.x**, LuaJIT / Lua 5.1 semantics.
- Upstream default branch is **`dev`**, not `main`.
- The mod platform lives in `src/mods/`; the reference book is the
  [project wiki](https://github.com/bryanthaboi/gen1recomp/wiki), and the registry reference is
  generated from `src/mods/Schemas.lua`.

You need a local checkout of the engine to build, run, or test anything here. Nothing in this
repo runs on its own.

## Dev loop

The loader discovers mods by scanning **`mods/` one level deep** in the engine checkout
(`src/mods/Loader.lua:_discover`). It does **not** walk `mods/examples/` — that is why gallery
entries ship disabled. Discovery explicitly tolerates `info.type == "symlink"`, so the standard
way to develop an out-of-tree mod is to symlink it in:

```sh
ln -s /Users/alamosaravali/Projects/alamops/RBYMMOMod /path/to/gen1recomp/mods/rby_mmo
```

Then, from the **engine checkout root**:

```sh
scripts/setup.sh --rom "/path/to/Poke Red.gb"   # one-time ROM import

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alamops/RBYMMOMod](https://github.com/alamops/RBYMMOMod) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
