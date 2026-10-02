---
trigger: always_on
description: Notes for anyone (human or agent) making changes here. Read this before
---

# Working on Pawvis

Notes for anyone (human or agent) making changes here. Read this before
touching the UI or the gesture engine — most of it exists because something
went wrong once.

## Build, test, run

```bash
swift build            # debug
swift test             # full suite — keep it green, it's the safety net
make app               # release build + assembles build/Pawvis.app + signs
open build/Pawvis.app
```

Extras:

- `build/Pawvis.app/Contents/MacOS/Pawvis --selftest` — headless smoke test
  (engine, settings round-trip, launch-at-login rules, voice parser).
- `PAWVIS_NO_AUTOSTART=1` — launch without starting tracking, so automated
  runs don't trip the camera permission prompt.
- `PAWVIS_OPEN_GUIDE=1` — open the Gesture Guide window at launch, the same
  eyes-on hook as `PAWVIS_OPEN_SETTINGS` below. Its posed-hand art is
  bundle-only, so a bare `swift run` shows the SF Symbol fallbacks instead:
  look at the guide from `build/Pawvis.app`, not from the binary.
- `PAWVIS_OPEN_WELCOME=1` — open the first-run welcome window at launch
  regardless of the `Pawvis.firstRunCompleted` flag. A genuinely new install
  (flag unset, camera not yet granted) gets it automatically instead of
  auto-starting tracking; `PAWVIS_NO_AUTOSTART=1` suppresses that and leaves
  the flag untouched, so automated runs stay headless.
- `PAWVIS_OPEN_PRACTICE=<page>` — open the practice round at launch on a
  page: `intro` (or `1`), a lesson name (`takeControl`, `move`, `click`,
  `drag`, `scroll`, `rightClick`), or `done`. The eyes-on hook for its
  lessons; it never touches the one-shot `Pawvis.practiceSeen` flag, which
  only the welcome tour's Start button sets.
- `PAWVIS_PRACTICE_DEMO=<none|found|armed|grabbed|dragging|scrolling|right>`
  — feed the practice window a synthetic hand in that state instead of the
  camera, so a screenshot machine with no hand in front of it still shows
  the real coaching states (the hand mirror, the status line). Only the
  hand feed is faked: the board itself still completes on real mouse
  events, so drive it with synthetic CGEvents like any other window.
- `Pawvis --gesture-eval <video…> [--verbose]` — run the real Vision +
  engine pipeline over a webcam recording and print every custom gesture
  that fires (plus per-frame openness/splay/palm diagnostics with
  `--verbose`). The ground-truth harness for the motion gestures: record a
  clip of the gesture and ask the machine, because synthetic tests cannot
  tell you what Vision does to a real hand mid-swipe. Every threshold in
  `CustomGestureDetector` was tuned against such clips; retune the same way.
- `Pawvis --attention-eval <video…> [--sensitivity 0…1] [--verbose]` — run
  the real face-detection + attention-gate pipeline (look-to-control) over a
  recorded clip and print every pause/resume transition (per-sample head
  yaw/pitch with `--verbose`). Same reasoning as `--gesture-eval`: what
  Vision reports for a real head mid-turn is a question for the machine.
- `PAWVIS_OPEN_THEREMIN=1` — open the theremin window at launch, switched
  off (no camera, no sound). With `PAWVIS_THEREMIN_DEMO=<playing|recording|
  take>` the window switches itself on and plays a synthetic two-handed
  phrase, with the camera never opened and the speakers muted (the
  recording tap sits upstream of the mute, so a demo take is real audio):
  the eyes-on hook for the instrument on a machine with no hand in front of
  it. `recording` starts a take half a second in; `take` stops it five
  seconds later so the strip shows a finished take.
- `PAWVIS_APPEARANCE=<light|dark>` — pin the app's appearance for the run,
  so both modes can be looked at without touching the system setting.
- `Pawvis --mp3-encode <audio file> <out.mp3> [bitrate]` — encode any file
  AVFoundation can read with the app's own MP3 encoder. The ground-truth
  harness for the encoder: decode the result with something that shares no
  code with it (ffmpeg, a browser, a phone) and compare. See
  [The theremin](#the-theremin).
- `Pawvis --cameras [uniqueID]` — list every camera macOS offers the binary,
  typed the way `CameraSelectionPolicy` sees them, and where Automatic (or
  the given pick) lands. Run it from `build/Pawvis.app/Contents/MacOS/Pawvis`:
  the Continuity Camera typing needs the bundle's Info.plist (see
  [Cameras](#cameras)).
- `Pawvis --action-eval <kind> [argument…]` — perform one gesture action
  through the real `GestureActionRunner` and print the pill outcome.
  "Does desktopRight actually switch the desktop on this machine" is a
  question for the machine, not for reading the code.
- `VERSION=1.2.3 BUILD_NUMBER=42 make app` — stamp a version into the bundle
  (CI does this from the release tag; local builds show `0.0.0-dev`).

**Signing matters more than it looks.** macOS ties the Accessibility grant to
the app's *designated requirement*. Signed with a real identity, that
requirement is identity-based and stable:

    identifier "com.pawvis.Pawvis" and anchor apple generic and ... leaf[subject.OU] = KMZ785G889

Ad-hoc signed, it is a per-binary `cdhash` instead — so every build looks like
a different app, macOS silently ignores the existing grant *while still
showing Pawvis as enabled*, and the symptom is "the cursor moves but nothing

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alexandriax/pawvis](https://github.com/alexandriax/pawvis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
