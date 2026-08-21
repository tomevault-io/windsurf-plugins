---
trigger: always_on
description: Abendrot is a macOS menu-bar app that warms the screen. It ships a small command-line
---

# Controlling Abendrot from an AI assistant or the terminal

Abendrot is a macOS menu-bar app that warms the screen. It ships a small command-line
tool, **`abendrot`**, that an AI assistant (Claude Code, Codex, Cursor) or a shell script
can use to read the app's live state and change its settings. This file is the canonical
reference for that control surface. Everything here is grounded in the real binary
(`abendrot --version` → `1.0.2`); no flag is described that the CLI does not implement.

This is a **capability** document: it describes what the `abendrot` CLI can do. It makes no
health or circadian claim — it only documents how to drive the app's visual state.

## TL;DR for an agent

```sh
abendrot set warmth 0.8      # set global warmth (0.0 = none … 1.0 = warmest)
abendrot status --json       # read live state back as JSON, then verify your change
```

`set warmth 0.8` validates the value, writes it to the app's preference domain (so it
survives a restart), and — if the app is running — applies it live and waits for the app to
confirm. `status --json` prints the live snapshot, including `globalWarmthStrength`, so you
can confirm the value landed. Full value ranges, the JSON field reference, and the exit-code
table are below.

If `abendrot` is not on your `PATH`, the binary ships inside the installed app bundle at
`/Applications/Abendrot.app/Contents/Helpers/abendrot` — invoke it by that full path (the
Homebrew cask symlinks it onto `PATH` for you). If neither is available, use the raw
`defaults` fallback in the appendix at the end of this file.

### Building / locating the CLI from source

If you are working in the source tree (not an installed app), build the CLI and run it from the
package's build directory rather than the app bundle:

```sh
swift build -c release --package-path cli
# binary lands at:
cli/.build/release/abendrot          # SwiftPM's stable per-config path (resolves to the arch dir)
./cli/.build/release/abendrot --version   # → 1.0.2
```

The CLI is a standalone SwiftPM executable; it controls the same running app via the shared
preference domain and `state.json`, so a source-built `abendrot` drives an installed/running app
exactly like the shipped one.

## How it works (one paragraph)

The CLI is a thin same-user client. A `set`/`on`/`off`/`exclude` command does two things:
(1) it writes the new value to the app's CFPreferences domain `app.abendrot.Abendrot`, which
persists across launches, and (2) it posts a local `DistributedNotification` that wakes the
running app so the change applies live. The app then writes a snapshot file
(`~/Library/Application Support/Abendrot/state.json`) stamped with the request id, and the
CLI polls that file to confirm the change was applied. `status` reads that same snapshot.
There is **no** daemon, socket, XPC service, network listener, or privileged helper.

---

## Commands

Every data-emitting command accepts `--json` for machine-readable output. `abendrot --help`
and `abendrot help <subcommand>` print the same surface this section documents.

### `abendrot status [--json]`

Read live app state: enabled, schedule mode, warmth (strength + approximate Kelvin),
reveal mode, the per-app exclusion set, and the per-display method actually in use. When the
app is **not** running, `status` falls back to the last-saved (persisted) values and says so.

```sh
abendrot status
# Abendrot 1.0.2 (build 4) — running
# Enabled: yes
# Mode: always-on (warming now)
# Warmth: 0.80 (~700K, max 500K)
# Reveal: hold
# Displays:
# • Built-in Retina Display: gamma
# • LG ULTRAFINE: gamma

abendrot status --json   # see the JSON field reference below
```

### `abendrot get <key> [--json]`

Print one configured (persisted) setting. Works whether or not the app is running, because
it reads the preference domain directly.

`<key>` is one of: `warmth` | `mode` | `max-warmth` | `cozy` | `reveal-mode` | `location` | `enabled`

```sh
abendrot get warmth          # 0.80
abendrot get mode            # always-on
abendrot get location        # auto    (or "37.77 -122.42" when set manually)
abendrot get enabled --json  # {"enabled":true}
```

### `abendrot on [--json]` / `abendrot off [--json]`

Enable or disable warming (`isEnabled`). This is the master toggle, independent of the
schedule mode.

```sh
abendrot on
abendrot off
```

### `abendrot set warmth [<strength>] [--kelvin <kelvin>] [--json]`

Set global warmth either as a strength `0.0`–`1.0`, **or** by targeting an effective Kelvin
with `--kelvin` (the CLI maps the Kelvin to a strength against the configured warmest-point
curve, exactly as the app would). Provide a strength **or** `--kelvin`, not both.

```sh
abendrot set warmth 0.8          # strength 0.0–1.0
abendrot set warmth --kelvin 3000  # target ~3000K effective
```

- `<strength>`: Double, `0.0`–`1.0`. Out-of-range input is rejected (exit 2), not clamped.
- `--kelvin <kelvin>`: Int, `500`–`6500`. Out-of-range rejected (exit 2).
- Giving **both** a strength and `--kelvin` is rejected (exit 2) — pick one.
- A bare leading-negative strength like `set warmth -1` is read as an unknown option and rejected
  (exit 2). To pass a negative value positionally, use the `--` terminator: `set warmth -- -1`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [matthewrball/abendrot](https://github.com/matthewrball/abendrot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
