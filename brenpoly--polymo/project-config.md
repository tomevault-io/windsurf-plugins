---
trigger: always_on
description: The product is **PolyMO** (renamed from "Digital Pet", 2026-08-27). The repo
---

# PolyMO

The product is **PolyMO** (renamed from "Digital Pet", 2026-08-27). The repo
directory, the Android package `com.digitalpet` and the firmware's internal
symbols are deliberately unchanged: renaming the package would be a different
install, losing models, pairing and transcript.

A physical digital pet: an **ESP32-S3 Waveshare Touch-AMOLED-1.8** board running
LVGL, paired over BLE to an **Android app** that does all the thinking on-device
(Whisper → llama.cpp → Piper). You tap the pet, speak at the pet, and the pet
answers out of its own speaker. The phone is compute; it is not a participant.

## Read these first

| Document | What it is |
|---|---|
| [`DESIGN.md`](DESIGN.md) | Product and UX: visual identity, flows, component rules, and **§7, the round trip with Claude Design** — read §7.3 before editing a mock. |

## Layout

```
pet-esp32/      ESP-IDF firmware for the current pet — the display, mic and speaker
android/        the app: BLE, Whisper, llama.cpp, Piper, and the conversation engine
design-system/  vendored copy of the Claude Design files — the sync tests read
                it. tokens/, strings.txt, components.txt, plus faces/ (what the
                pet LOOKS like) and personas/ (what it SAYS). Code is GENERATED
                from the last two — see below.
tools/          design-sync.sh (the front door), gen-faces.py, gen-personas.py,
                design-remote.py   (serlog.py lives in pet-esp32/tools/)
shared/         the wire protocol header both sides build against
DESIGN.md       product/UX
```

The git root is this directory, not `pet-esp32/`. Firmware and app share a wire
protocol (`main/pet_proto.h` ↔ `ble/PetProtocol.kt`) and must change in step.

## Build

```bash
# firmware
. ~/esp/esp-idf/export.sh
cd pet-esp32 && idf.py build
idf.py -p $(ls /dev/cu.usbmodem* | head -1) flash   # port name changes on re-plug

# android — MUST use JDK 21; the system JDK is 25 and Kotlin 2.1 cannot parse it
cd android
JAVA_HOME="/Applications/Android Studio.app/Contents/jbr/Contents/Home" \
  ./gradlew assembleDebug --offline
```

Tests: `./gradlew testDebugUnitTest --offline` (512, no device needed).
**Write them by mutation** — every test here was checked by breaking the code it
covers and confirming it fails. That practice has already caught two worthless
tests in this repo. The second was a `ContrastTest` assertion
that read the palette file instead of the theme, so rewiring a colour role left
it green.

Most are pure logic. Five are not, and they are the ones that catch what a build
cannot — all five read files rather than calling functions:

| | |
|---|---|
| `ContrastTest` | WCAG ratios over both colour schemes, measured on the schemes themselves |
| `PetLiteralsTest` | scans UI sources, fails on a raw `.dp` or `.sp` |
| `TokenSyncTest` | every colour role, token, radius and type style against `design-system/tokens/` |
| `ComponentRosterTest` | `ui/components/` against `design-system/components.txt` |
| `FaceSyncTest` | the compiled face sets against `design-system/faces/*.json` |

They exist because a visual property that is *arithmetic* never needed eyes.
**`design-system/` and `ui/` are declared test inputs in `app/build.gradle.kts`** —
without that, Gradle skipped the test task as UP-TO-DATE whenever only those
files changed, which is the one case the sync tests exist for.

## Conventions worth knowing before changing anything

- **Verify on hardware, not by building.** Nearly every hard bug in this project
  looked fine in a build and wrong on the board. Record the measurement, not
  just the fix.
- **Commit messages carry the *why*.** They are long here on purpose and are a
  primary record. Write them with `git commit -F <file>` — backticks in
  `-m "…"` get executed by the shell and silently delete text.
- **A comment that is now false is worse than no comment.** Several bugs here
  were prolonged by confident, stale notes. If a change inverts one, fix it in
  the same commit.

## The pet's face and voice are GENERATED — do not hand-edit them

`design-system/faces/*.json` is the source for the firmware's `pet_faces.h`, the
app's `PetFaceSets.kt`, 32 notification drawables and four design-system cards.
`design-system/personas/*.json` is the source for `PetPersonas.kt`, which holds
both the LLM system prompt and every line the pet says without being asked.

```bash
tools/gen-faces.py        # after editing a face set
tools/gen-personas.py     # after editing a persona
```

Both refuse input that breaks a rule, and say which: a happy face must smile, a
dead one has no mouth, sad and sick must not be confusable, a spiral eye must
turn and must not wind tight enough to fill in, a persona may not ask a question
or name several apps aloud, and nothing may restate a `PET_RULES` line. **Those rules are the design, expressed where they can fail a build** —
read the two READMEs before adding a set.

Three things that are NOT a persona's to change, because they belong to the pet
rather than to a character: the reply-length cap (the screen fits six lines), the
no-history rule (no history is sent, so without it the model invents callbacks),
and staying in character. They are appended to every prompt automatically.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [brenpoly/polymo](https://github.com/brenpoly/polymo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
