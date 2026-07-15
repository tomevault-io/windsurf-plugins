---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build

Run from the `dealers_choice/` directory:

```sh
meson setup _build_dealers_choice --reconfigure -Dgen_protobuf=true
meson compile -C _build_dealers_choice
```

All `meson compile`, `meson test`, and `meson devenv` commands must use `-C _build_dealers_choice`. Never use `_build` or `build`.

To update subprojects after upstream changes:

```sh
meson subprojects update
```

## Tests

**See [`tests/README.md`](tests/README.md) for the full testing guide** — the
meson suite (unit + game-logic groups), the `scripts/` harnesses and
`scripts/soak.sh`, sanitized builds for live verification, `tc netem` impairment,
heaptrack profiling, running the binaries by hand (data-dir / bot-password /
`gen_protobuf` gotchas), screenshotting the GUI, and troubleshooting.

```sh
meson test -C _build_dealers_choice -v
```

Quick reminders (details in `tests/README.md`):
- All `meson` commands use `-C _build_dealers_choice` (see Build); never
  `_build` / `build`.
- The CI gate is **gcc `-Werror`**, but the ASan build is **clang** — do a gcc
  pass before calling a branch CI-ready (clang stays silent on
  `-Wformat-truncation` and the like).
- After **server-side** changes, run a short `scripts/soak.sh` and judge it by
  the log (`DONE: all phases passed`), not the exit code.

### Claude Code commands

Project slash commands live in `.claude/commands/`. They are discovered **only
when Claude Code is launched from the repo root** (`dealers_choice/`) — starting
from a parent dir means `/dc-*` won't resolve (`cd`-ing in afterward doesn't fix
it; the project root is fixed at launch). Available:

- `/dc-verify` — gcc `-Werror` gate + clang ASan/UBSan build + full meson tests.
- `/dc-soak` — short (~5 min) sanitized bot soak (run after server-side changes).
- `/dc-gate` — pre-merge gate: runs `/dc-verify` then `/dc-soak`, one PASS/FAIL.

## Design ethos (scope)

DC is a **casual** game — for friends, family, and strangers — explicitly **not
competitive**. This steers which features are worth building:

- **No leaderboards or rankings.** They reward play-time over skill, which isn't
  fair when people can't all play as often.
- **Prefer session-scoped continuity over persistent accounts.** The real player
  need after a brief disconnect is to resume the current game with their stack,
  not a permanent cross-server account. Favour an ephemeral in-session reconnect
  (hold seat + chips for a grace window, rejoin via a per-session token) over a
  persistent global identity. A persistent pubkey identity is also a cross-server
  tracking handle, which cuts against the low-stakes spirit.
- When suggesting features, bias toward low-friction, social, session-scoped
  mechanics; **flag** anything that introduces ranking, persistence, or accounts
  rather than silently adding it.

## Architecture

**Client-server** networked game, split across three binaries that talk over
TCP: the GUI client (`dealers-choice`), a headless server
(`dealers-choice-server`), and a headless bot (`dealers-choice-bot`). The GUI is
a **client only** — it no longer hosts a server (the old `--server` mode is a
deprecation stub; see "headless server binary name" below). Sources live under
`src/{game,net,server,ui}/` by area, with the SDL-free logic in `libdc_core`.

### Static library chain

Meson builds the local libs bottom-up:

1. **tcpme_lib** (`src/tcpme/`) — cross-platform TCP socket abstraction (POSIX + Winsock). Exposed as `tcpme_dep` and merged into `shared_dep`. Note: tcpme was written almost entirely by an LLM, which is a potential concern for correctness and security.
   - **Working on tcpme? See the `tcpme` skill** (`.claude/skills/tcpme/`): the design contract (keep it generic — *mechanism*, not *policy*; expose a knob and let DC own the value, e.g. `tcpme_set_timeout(ms)` with DC's `SOCKET_IO_TIMEOUT_MS`), the POSIX↔Winsock traps, and an audit checklist for the LLM-written, internet-facing code.
2. **`libdc_core`** (`dc_core_dep`) — the **SDL-free** core: game rules, protocol/serialization, and the server engine (`common_src` = `src/{game,net,server}` + core root files). No SDL, ttf, image, or audio. Linked by all three binaries.
3. **`_lib` ("game")** (`game_dep`) — the GUI library: rendering, widgets, menus (`gui_src` = `src/ui/` + `widgets_src`), adding SDL2/ttf/image + miniaudio on top of `dc_core_dep`.

Binaries: `dealers-choice` = `main.c` + `game_dep`; `dealers-choice-bot` = `bot.c` + `dc_core_dep`; `dealers-choice-server` = `src/server/server_main.c` + `dc_core_dep`.

**Why the split:** the bot and server binaries must link **no SDL at all**, so anything pulling in SDL/ttf/image/audio belongs in `src/ui/` (the GUI lib), never in `libdc_core`. When adding a source file, decide: shared game/protocol/server logic → an area dir under `src/{game,net,server}`; GUI-only → `src/ui/` (`ui_src`) or `src/widgets/`.

### Key source files

| File | Role |
|---|---|
| `src/main.c` | GUI entry point: SDL init, top-level menu, connection loop |
| `src/cli.c` | Shared CLI parsing — `parse_cli_args` (client) + `parse_server_args` (server) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Dealer-s-Choice/dealers-choice](https://github.com/Dealer-s-Choice/dealers-choice) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-15 -->
