---
trigger: always_on
description: You're in the **mob** repo, the runtime library for the Mob mobile framework.
---

# AGENTS.md — orientation for AI agents working on Mob

You're in the **mob** repo, the runtime library for the Mob mobile framework.
Read this in full before making changes — it's the 5-minute orientation that
will keep you from re-deriving things the rest of the team has already learned
(or learned the hard way).

## What Mob is, in one paragraph

Mob lets you write iOS and Android apps in Elixir, with the BEAM running
on-device. The phone hosts an Erlang node — a real one, distribution-capable,
introspectable, hot-code-loadable. Two modes: a SwiftUI/Compose UI driven by
Elixir GenServers (Mob UI apps), or a sidecar BEAM embedded in a normal native
app to give agents and tests live access (Mob as test harness). The sidecar
mode is the long-term bet. Both modes produce a real Erlang node you can `Node.connect/1` to.

For the *why* (the BEAM-on-mobile pitch), see `guides/why_beam.md`.

## Repo topology

Mob is three coordinated repos. **Know which one to edit before you change anything.**

| Repo | Path | What lives here | Edit when |
|---|---|---|---|
| **mob** | `~/code/mob` | Runtime library: `Mob.Screen`, `Mob.App`, `Mob.Renderer`, `Mob.Dist`, `Mob.Test`, the iOS Swift / Android Kotlin native bridges, the NIF | UI behavior, on-device runtime, native bridge changes |
| **mob_dev** | `~/code/mob_dev` | Mix tasks: `mob.deploy`, `mob.connect`, `mob.devices`, `mob.emulators`, `mob.provision`, `mob.doctor`, `mob.battery_bench_*`. Igniter installers (`mob.add_nif`, `mob.enable`, `mob.adopt`). Device discovery (`MobDev.Discovery.{Android,IOS}`). Native build orchestration (`MobDev.NativeBuild`). OTP tarball download/cache (`MobDev.OtpDownloader`). | Build/deploy mechanics, device handling, dev tooling, **Igniter tasks that mutate an existing project** |
| **mob_new** | `~/code/mob_new` | Project generator. Hex archive (`mix archive.install hex mob_new`). Templates in `priv/templates/mob.new/`. Generates both native Mob UI projects and Phoenix LiveView wrappers. | Greenfield generator output. **Must stay self-contained** (`ArchiveSelfContainedTest`) — no hex-dep modules reachable from archive code, so Igniter-based tasks live in mob_dev, not here |

Cross-repo changes are common — fixing one user-visible behavior often needs
the runtime patched in `mob`, the build retooled in `mob_dev`, **and** the
generator template updated in `mob_new` so newly-generated projects pick up
the fix without manual edits.

The OTP runtime tarballs (Android arm64/arm32, iOS sim, iOS device) are built
separately and uploaded to GitHub Releases — see `mob_dev/build_release.md`
and `mob_dev/scripts/release/`. Patches we apply to OTP source live at
`mob_dev/scripts/release/patches/`.

## Driving apps from your session

The default instinct — screenshots — is wrong. Mob apps run a real Erlang node
you can talk to directly. Read the BEAM, drive it, then verify visually only
when state isn't enough.

### Connect

```bash
mix mob.devices                 # list everything connected (sims, emulators, physical)
mix mob.emulators --list        # list virtual devices (running and stopped)
mix mob.connect                 # set up tunnels, start IEx attached to all running nodes
mix mob.connect --no-iex        # just print node names + tunnels (for scripting)
```

Node names are platform-specific:

```
mob_demo_ios@127.0.0.1                     # iOS simulator
mob_demo_android_<serial-suffix>@127.0.0.1  # Android (suffix from ro.serialno)
```

For iOS simulator, the sim shares the Mac's network stack — distribution Just
Works. For Android (and iOS device), `mix mob.connect` sets up `adb reverse` /
similar tunnels.

### Inspect (`Mob.Test`, BEAM-state, fast, exact — prefer this)

```elixir
node = :"mob_demo_ios@127.0.0.1"

Mob.Test.screen(node)            # which screen is showing?  → ModuleName
Mob.Test.assigns(node)           # live socket assigns        → %{...}
Mob.Test.find(node, "Submit")    # locate widget by visible text
Mob.Test.inspect(node)           # full snapshot: screen, assigns, nav stack, widget tree
```

This is faster, exact (not pixel-inferred), and works without taking a
screenshot. Use it as the default.

### Drive

```elixir
Mob.Test.tap(node, :open_text)              # tap by tag atom (the on_tap: {self(), :tag})
Mob.Test.send_message(node, {:custom, :msg}) # arbitrary handle_info
```

After a tap, call `Mob.Test.screen(node)` again to confirm navigation
happened. Call `Mob.Test.assigns(node)` to confirm state changed.

### Visual verify (MCP, slower, image-based — only when needed)

When layout/animation/rendering matters, fall back to MCP platform tools:

| iOS simulator | Android |
|---|---|
| `mcp__ios-simulator__screenshot` | `mcp__adb__dump_image` |
| `mcp__ios-simulator__ui_view` | `mcp__adb__inspect_ui` |
| `mcp__ios-simulator__ui_tap {x, y}` | `adb shell input tap` |
| `mcp__ios-simulator__ui_swipe` | `adb shell input swipe` |
| `mcp__ios-simulator__record_video` | `adb shell screenrecord` |

Use these to confirm a layout looks right, spot animation glitches, or
debug rendering. **Don't use them for state queries** — `Mob.Test.assigns/1`
is always better.

### Round-trip workflow

```
1. Edit Elixir/Swift/Kotlin code
2. mix mob.push                  # fast: BEAM-only push, no native rebuild

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GenericJam/mob](https://github.com/GenericJam/mob) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
