---
trigger: always_on
description: macOS menu bar utility that remaps the three buttons on Apple's **wired**
---

# AirPodKit — dev notes

macOS menu bar utility that remaps the three buttons on Apple's **wired**
EarPods/AirPods remote (Volume Up, center/Play-Pause, Volume Down) to
custom keyboard shortcuts. SwiftUI + AppKit, macOS 13+, no App Sandbox,
signed with a local dev certificate (not Developer ID yet — see Signing
below). Original design/plan lives at
`~/.claude/plans/stateful-dancing-nest.md` if it still exists on this
machine; treat this file as the up-to-date source of truth instead.

## Build & run

```
./build.sh          # regenerates AirPodKit.xcodeproj via xcodegen, builds, copies to ./build/AirPodKit.app
./build.sh --run    # same, then kills any running instance and relaunches
```

`AirPodKit.xcodeproj` is generated from `project.yml` by `xcodegen` (`brew
install xcodegen`) — it is gitignored, never hand-edit it, edit
`project.yml` instead. Source files are added automatically since
`project.yml` just globs the `AirPodKit/` directory.

`./build/AirPodKit.app` is the stable, fixed path the app always builds
to. Don't move it — see Signing/TCC below for why the path matters.

## Architecture

```
AirPodKitApp (SwiftUI @main)
  └─ MenuBarExtra(.window) → PopoverRootView   (the UI, menu bar popover)
  └─ AppDelegate (NSApplicationDelegate)        (real onboarding window, see below)

RemapEngine.start()  — called once from AirPodKitApp.init()
  ├─ wires RemoteButtonMonitor.onButtonPress → looks up ShortcutStore →
  │    KeystrokeSynthesizer.post(...) if mapped, else lets the event through
  └─ subscribes to PermissionsManager's published grant state and
       starts/stops RemoteButtonMonitor's CGEventTap accordingly (not just
       once at launch — see "Permission flow" below, this is the thing
       that was actually broken and fixed twice)

RemoteButtonMonitor  — the CGEventTap. Owns the tap lifecycle + health check.
KeystrokeSynthesizer — posts synthetic CGEvents for the configured shortcut.
Shortcut / ModifierKey — the data model (see below).
ShortcutStore        — Codable JSON in UserDefaults, ObservableObject.
PermissionsManager   — polls + requests Accessibility / Input Monitoring.
LaunchAtLoginManager — thin wrapper over SMAppService.mainApp.
NowPlayingClaim      — optional compatibility experiment; not activated by
                        default because AirPodKit is not a media player and a
                        global claim can steal ordinary media controls.
DebugLog             — appends timestamped lines to
                        ~/Library/Logs/AirPodKit/airpodkit.log asynchronously;
                        use this to debug button handling instead of stdout
                        prints, since the app usually has no attached terminal.
```

### How button presses are captured

Wired remote buttons surface on macOS as the same private event stream
used for keyboard media keys: `NSEvent.systemDefined` (type 14, aka
`NX_SYSDEFINED`), `subtype == 8` (`NX_SUBTYPE_AUX_CONTROL_BUTTONS`), with
the key code and down/up state packed into `data1`:
- high 16 bits = key type: `NX_KEYTYPE_SOUND_UP=0`, `NX_KEYTYPE_SOUND_DOWN=1`,
  `NX_KEYTYPE_PLAY=16` (center button — mapped to Play/Pause's key type)
- bits 8–15 = state: `0x0A` = down, `0x0B` = up

`RemoteButtonMonitor` installs a `CGEventTap` at `kCGHIDEventTap` with
`kCGEventTapOptionDefault` (an active/consuming tap, not listen-only),
decodes this, and if `ShortcutStore` has a mapping for that button, returns
`nil` from the callback (swallows the event, so the system doesn't change
volume / trigger play-pause) and posts a synthetic replacement keystroke.
If unmapped, the event is returned unmodified (`Unmanaged.passUnretained`,
**not** `passRetained` — the callback doesn't own a retain on the event it's
handed, this was a real bug once, see git history) so default behavior
happens.

⚠️ **Must consume both the down AND up half of a mapped press, not just
down.** Each physical press generates two `NX_SYSDEFINED` events (down
then up). Originally only the down event was inspected/consumed and the
up event was unconditionally passed through — the center button mapped to
a shortcut fired the shortcut correctly on down, but the untouched up
event still reached the system and triggered Play/Pause anyway, since
macOS's default handling for that key type acts on the up transition. Fix:
`RemoteButtonMonitor.onButtonPress` is now called for *both* halves
(signature is `(RemoteButton, isDown: Bool) -> Bool`) and consumes both
when mapped. `RemapEngine` normally calls `KeystrokeSynthesizer.post` on
down, then consumes up; if macOS delivers an up without a paired down, it
synthesizes once on up as a fallback and tracks the press so a normal pair
still fires only once.

✅ **Confirmed working against the user's real wired remote.** The
throwaway diagnostic spike (`Spike/main.swift`, still present, unbuilt
binary gitignored) never actually captured a press during its own test
session, so the mechanism went unverified for a while — but combo
shortcuts (modifier + base key) have since been confirmed end-to-end on
real hardware: pressing the physical button fires the mapped shortcut and
the original volume/play-pause action is suppressed. If a *future* piece
of hardware (different cable/dongle) doesn't work, this is still the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wdwxw/airpodKit](https://github.com/wdwxw/airpodKit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
