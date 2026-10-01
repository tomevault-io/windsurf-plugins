---
trigger: always_on
description: Instructions for AI coding agents working in the Falcon repository.
---

# AGENTS.md

Instructions for AI coding agents working in the Falcon repository.

If you are a human, read [CONTRIBUTING.md](CONTRIBUTING.md) instead. It is the authoritative document, and
where the two disagree, CONTRIBUTING.md wins.

---

## The project

Falcon is a Minecraft: Bedrock Edition server written from scratch in C++. It is not derived from any other
server: the protocol, storage, gameplay and networking are implemented in this organisation's repositories.

- **Language:** C++17. Do not use newer language features.
- **Build system:** CMake with Ninja, dependencies pulled with `FetchContent`.
- **Platforms:** Windows (MSYS2 UCRT64, gcc), Linux (gcc) and macOS (clang), all built by CI.
- **License:** LGPL-3.0.
- **Branch:** `main`. Protocol upgrades are prepared on a branch named after the upcoming game version.

---

## Required reading

| File                               | Read it for                                                        |
|------------------------------------|--------------------------------------------------------------------|
| [CONTRIBUTING.md](CONTRIBUTING.md) | **Mandatory.** Contribution rules, AI disclosure, closing reasons. |
| [SECURITY.md](SECURITY.md)         | What counts as a vulnerability and how to report one privately.    |
| [README.md](README.md)             | Project overview and supported versions.                           |

---

## Commands

| Task                          | Command               |
|-------------------------------|-----------------------|
| Configure and build           | `./build.sh`          |
| Configure and build (Windows) | `build.bat`           |
| Incremental rebuild           | `cmake --build build` |

The first configure downloads dependencies and needs network access. Build output goes to `build.txt`. A
change that does not compile is worse than no change.

---

## Layout

```
Falcon.Server/
  include/                      Headers, mirroring src/
  src/
    Network/Handler/            Session, login, movement, inventory, damage, commands
    Level/                      Worlds, chunks, LevelDB storage, world generation
    Block/Blocks/               One class per block family
    Block/Systems/              Redstone, pistons, fluids, fire, random ticks, copper...
    Item/Items/                 One class per item behaviour
    Actor/                      Players, mobs, projectiles
    Actor/AI/Goal/              Mob AI goals
    Command/                    One class per command
    Scripting/                  JavaScript scripting API for behavior packs
```

The network transport, protocol packets, NBT and Bedrock data files live in separate `Falcon-MC`
repositories. Never edit their copies under `build/_deps`.

---

## Hard rules

1. **Never invent APIs.** Before calling a function, field, packet, block state or registry entry, search the
   codebase and confirm it exists with that exact signature. If you cannot find it, say so.
2. **Never guess vanilla behaviour.** Packet formats, constants, tick rates, chances and state names must
   match the game. If you cannot verify a value, say which one.
3. **One logical change per branch.** Report unrelated problems you notice instead of fixing them.
4. **No repository-wide reformatting** and no refactors outside the task.
5. **No new dependencies** without an issue first.
6. **No dead code** and no debug logging left in the diff.
7. **Do not weaken security checks**: authentication, packet validation, bounds checks, rate limits. If one is
   in your way, stop and explain.
8. **Do not touch** `.github/workflows/` or release automation unless that is the task.
9. **Do not commit, push or open a pull request** unless the human explicitly asks.

---

## Architecture

- **Classes, not string checks.** Blocks, items and actors with behaviour are classes registered with
  `FALCON_REGISTER_BLOCK`, `FALCON_REGISTER_ITEM` or `FALCON_REGISTER_ACTOR`. The type is decided once, in the
  class `matches()`. Systems call virtual methods and never compare identifiers to find out what something is.
- **Generic over specific.** Prefer `BlockActor::getContainer()` to a chest-only accessor. A function whose
  name contains a concrete type usually has a generic form.
- **Damage.** All player damage goes through `ServerNetworkHandler::hurt` with a `DamageSource`. It handles
  gamerules, the invulnerability window, shields, armor, effects and the totem.
- **Threads.** Game state belongs to the main thread. Chunk workers and the network thread only exchange
  data through queues. Never read or write `Level` chunks or players from a worker.
- **Placement.** `Level::setBlock(position, state, true)` updates the block and its neighbours. Player
  placement does not run the placed block's own update: override `onPlaced` when a block must react to being
  placed.
- **Waterlogging** is block layer 1. `Level::peekBlockPtr` returns `nullptr` for chunks that are not loaded:
  treat that as unknown, not as air.

---

## Best practices

- **Match the surrounding file.** Consistency with neighbouring code beats your preferred style.
- **Search before writing.** Look for an existing helper or system first. A duplicated helper is a review
  comment every time.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Falcon-MC/Falcon](https://github.com/Falcon-MC/Falcon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
