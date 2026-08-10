---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A Fabric mod (Java 21) that runs inside Minecraft's JVM. It hides the vanilla HUD (health, hunger, XP, armor, hotbar) and streams that state every client tick over a loopback TCP socket, so a separate companion Android app can render it on a second screen. It also accepts short text commands back over the same socket (hotbar slot selection, simulated keybind presses, item-icon requests).

This mod is one half of a two-process system targeting the **AYN Thor** (dual-screen Android handheld) running Minecraft via **Zalith Launcher 2**. The other half — the Android companion app that renders the HUD — lives in a sibling repo, `Android_AynThor_MinecraftSecondScreen`. The two communicate only via the socket protocol; there's no shared code or build.

**`thor-hud-handoff.md`** is a detailed technical handoff/design doc covering the full system (both repos) — read it first for the JSON protocol spec, architecture rationale, and known issues. It's a snapshot from a point in development and may drift from the current tree in file/class specifics (there was a period of frequent package renaming) — verify anything file-specific against the actual source before relying on it.

## Commands

```
./gradlew build          # compile + package the mod jar
./gradlew runClient      # launch a dev Minecraft client with the mod loaded
```

Requires JDK 21. Uses Fabric Loom with `splitEnvironmentSourceSets()` enabled, so code lives in either `src/main` (both sides) or `src/client` (client-only — nearly all of this mod's logic lives here, since it's a client-only HUD mod). Versions (Minecraft, Yarn mappings, Loader, Fabric API) are pinned in `gradle.properties` — check there before assuming an API shape, since Yarn mappings have changed across versions used during development and a number of API calls in this codebase were written from best recollection rather than confirmed against the mappings (see `thor-hud-handoff.md` §5).

Manual protocol smoke test (no Android app needed): run `runClient`, join a world, then from another terminal:
```
nc localhost 48291
```
You should see one JSON line per tick.

Scripted version, which is what to actually use — it auto-joins a world and dumps the whole protocol (assets, a HUD snapshot, map PNGs, bindings, chat) to disk:
```
./gradlew runClient -PquickPlay="New World"
python tools/capture_protocol.py --out captures
python tools/capture_protocol.py --out captures --say "hello"   # chat round trip
```
`--say` sends a `CHAT:` command and waits for the server's echo to come back as a chat line, which exercises both directions in one run. A leading `/` makes it a command, as in game.

## Architecture

Full protocol spec (message shapes, field meanings, command codes) is in `thor-hud-handoff.md` §2 — don't re-derive it from source when the doc already has it verified. In short: the mod broadcasts a full HUD snapshot (health/food/xp/armor/hotbar) every client tick (~20/sec, unthrottled) over `127.0.0.1:48291`, newline-delimited JSON. It replies to `ICON:<itemId>` requests with a base64 PNG, and accepts single-tick simulated key/hotbar-press commands as plain text lines.

**Why a socket, not shared memory:** the mod and the companion app run in separate Android app sandboxes (different processes/UIDs) despite being on the same device, so loopback TCP is the simplest channel available.

### Key classes (`src/client/java/com/exojosh/client/`)

- `GameHudVisibility` — decides per element whether the *game* draws its own HUD. **This is why the mod no longer calls `HudElementRegistry.removeElement()`:** removal is a one-way door, since the vanilla element's renderer belongs to vanilla and nothing is kept that could put it back. `replaceElement()` hands over the original and takes a wrapper that decides per frame whether to delegate; with nothing shown, that's behaviourally identical to the old removals. The companion app drives it — an element switched off on the second screen is handed back to the top screen, so between the two displays the HUD is drawn exactly once. Wire format is `HUD:<key>,<key>,…`, the **full set every time**, re-sent on every toggle and reconnect: a dropped or reordered delta would leave the two screens disagreeing about who owns an element, which shows up as one drawn twice or not at all. Keys must match `HudElement.key` in the app's `settings/HudSettings.kt`; `offhand` deliberately has no entry here, because vanilla draws the off-hand box as part of the hotbar.
- `ThorHudClient` — `ClientModInitializer`; installs `GameHudVisibility`'s wrappers, starts `HudStateServer`, and on each `ClientTickEvents.END_CLIENT_TICK` builds/broadcasts a `HudState` and drains inbound commands (routing `ICON:`-prefixed ones separately from action commands). `pushBundleToNewClients()` sends each new connection the texture bundle then the key bindings; both have to run on the client thread (resource manager, `GameOptions`/language files), which is why they're here and not on the accept thread.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [exojosh/AynThorSecondScreen](https://github.com/exojosh/AynThorSecondScreen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-10 -->
