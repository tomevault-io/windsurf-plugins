---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

BFA-HavenCore — a TrinityCore-derived World of Warcraft server emulator targeting client **8.3.7 (build 35662)**. C++17, CMake ≥ 3.27. Two server binaries: `worldserver` (game) and `bnetserver` (Battle.net login). Open source under GPL-3.0 and community-maintained: changes arrive as pull requests from many contributors, so a change has to be legible to someone who was not in the room when it was written.

**This file is the repository-wide rule file.** It is tracked, it applies to every branch and every contributor, and it is **not** one of the AI planning artefacts covered by the hygiene rule below — never delete or gitignore it when cleaning a branch for merge.

### Current branch context

`feature/corruption-system` is developed in a separate clone of the community repo. The planning artefacts for that work — `docs/superpowers/` and `.superpowers/` — are clone-local and are deleted before the branch merges back; the community repo carries no AI planning docs. Design of record: `docs/superpowers/specs/2026-07-30-bfa-corruption-system-design.md`, with plans in `docs/superpowers/plans/`. Task ledgers live in `.superpowers/sdd/<plan-slug>/progress.md` — read the ledger before continuing an in-flight plan; it records deferred items and controller rulings that the commits do not.

## Build

Windows / Visual Studio 2022 is the primary toolchain. The configured tree is `./build` (VS 17 2022, x64, `TOOLS=0`).

```powershell
# configure (only needed after CMakeLists/option changes)
cmake -S . -B build -G "Visual Studio 17 2022" -A x64 -DTOOLS=0

# build one target — worldserver is the usual one
cmake --build build --config RelWithDebInfo --target worldserver

# everything
cmake --build build --config RelWithDebInfo
```

Targets: `worldserver`, `bnetserver`, `game`, `scripts`, `shared`, `common`, `database`, `proto`. Binaries land in `build/bin/RelWithDebInfo/`.

Useful options (`cmake/options.cmake`): `SCRIPTS` (`static` default / `dynamic` / `minimal-*` / `none`), `TOOLS` (map/vmap/mmap extractors, off here), `WITH_WARNINGS`, `WITH_COREDEBUG`, `NOPCH` (kills both PCHs). In-source builds are blocked by `CMAKE_DISABLE_IN_SOURCE_BUILD`.

Third-party deps are vendored under `dep/` (boost stub, CascLib, protobuf, recastnavigation, fmt, jemalloc, …). Host prerequisites: Boost 1.81, MySQL ≥ 8.0, OpenSSL 3.x.

Linux builds go through Docker; `docker/README.md` is the authoritative guide (`docker compose build`, `docker compose run --rm db-import`, `docker compose up -d bnetserver worldserver`). For an iterative C++ loop use the `dev-builder` override documented in `docker-compose.dev.yml`.

## Tests

**There is no test framework in this repository.** Every verification is either a build check or a stated manual in-game check. Do not claim a change is verified on the strength of a clean compile alone — say which of the two you actually ran, and if an in-game check is required and was not performed, say so explicitly.

## Architecture

### Layering
`src/common` (platform, logging, config, crypto, collision, threading) → `src/server/database` (MySQL pools) + `src/server/shared` (networking, realm, packet plumbing) + `src/server/proto` (Battle.net protobuf services) → `src/server/game` (all gameplay) → `src/server/scripts` (content scripts) → `src/server/{worldserver,bnetserver}` (entry points). `src/tools` holds the client-data extractors.

### Four databases
`auth` (login), `characters`, `world`, `hotfixes` — connection strings in `worldserver.conf` (`LoginDatabaseInfo`, `WorldDatabaseInfo`, …). All queries go through per-database prepared-statement enums in `src/server/database/Database/Implementation/{Login,Character,World,Hotfix}Database.h` (`CHAR_SEL_*`, `CHAR_UPD_*`, … terminated by `MAX_*DATABASE_STATEMENTS`), used as `CharacterDatabase.GetPreparedStatement(CHAR_UPD_ITEM_INSTANCE)`. Adding a query means adding the enum member *and* its `PrepareStatement` line in the matching `.cpp`. Async results come back via `QueryCallback` / query holders — never block a map thread on a query.

Migrations live in `sql/updates/world/YYYY_MM_DD_NN[_description].sql` and are applied at startup by `DBUpdater`/`UpdateFetcher` according to the `Updates.EnableDatabases` bitmask. The directory name is not free: `UpdateFetcher` reads it from the `updates_include` table, which the base dump seeds as `$/sql/updates/world`. Renaming the directory silently disables the updater — it logs one WARN and then finds no files at all. Base dumps (`sql/base/*.sql`) are gitignored and not part of the repo.

### Client data (DB2)
`src/server/game/DataStores/` mirrors the client's `.db2` files: `DB2Stores.cpp` (store declarations + load order), `DB2Structure.h` (row structs), `DB2LoadInfo.h` / `DB2Metadata.h` (field layouts that **must** match the client build), `DBCEnums.h` (constants and ID enums). Touching one of these usually means touching all of them. Row data is read from `DataDir` (`.\ClientData`), extracted from the 8.3.7 client — treat the DB2s as ground truth and never hardcode a value you could read from them.

### Opcodes and packets

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HavenWoW/BFA-HavenCore](https://github.com/HavenWoW/BFA-HavenCore) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
