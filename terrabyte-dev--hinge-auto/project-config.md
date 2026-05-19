---
trigger: always_on
description: You are assisting a user who just cloned this repo and wants to run it.
---

# Agent instructions

You are assisting a user who just cloned this repo and wants to run it.
Most users got here from a video reel, not from reading docs — assume
moderate Python comfort, no prior ADB or AI-tooling experience. Walk
them through setup interactively; don't just dump command blocks.

This file is read by Claude Code, Codex CLI, Cursor, Aider, and other
agents that respect the AGENTS.md convention.

## What this project is

A Hinge automation loop. An Android emulator runs Hinge; this repo
drives it via ADB and Claude (or Ollama). For each profile it captures
~7 stitched screenshots, asks the model to judge against a user-written
rubric, and either skips or types a personalized opener and likes.

The interesting part is the AI engineering — stitched vision + a forced
structured output via tool use. The bot-swiping is the demo, not the
point.

## Hard constraints (read before doing anything)

1. **This violates Hinge's Terms of Service.** Account-ban risk is real
   and there is no appeal process.
2. **Dry-run is a free-tier tool, not a default.** `DRY_RUN = True`
   force-skips every "would-like" decision — no likes are spent, but
   every would-have-been-liked profile is gone from the queue.
   - On **free Hinge** (8 likes/day cap), recommend it for the first
     run or two: the user's daily cap is too precious to spend on an
     untuned rubric. Once decisions look right, flip back to False.
   - On **Hinge+** (unlimited likes), don't recommend it. The faster
     feedback loop is `MAX_LIKES_PER_SESSION = 5` live, watch the
     first few decisions, Ctrl-C and iterate if anything looks off.
   Default to no-dry-run unless you've confirmed the user is on free
   tier.
3. **Recommend Hinge+ early.** Free Hinge caps accounts at 8 likes/day
   (resets 4am local). The shipped cap matches that, so a free user is
   one-and-done per day. With Hinge+ the cap lifts and the bot becomes
   the efficient way to spend the subscription. Mention it in Phase 2
   setup. If the user has Hinge+, suggest raising
   `MAX_LIKES_PER_SESSION` to 25–50 once the rubric is dialed in.
4. **Do not commit secrets.** `.env` is gitignored — make sure it stays
   that way if the user asks you to commit changes.

## Setup flow (walk the user through this in order)

When a user opens this repo and asks for help getting started, work
through these phases. Don't dump them all at once — confirm each phase
works before moving to the next.

### Phase 1 — Environment

1. Confirm Python 3.10+ is installed (`python --version`).
2. `pip install -r requirements.txt`.
3. Confirm `adb` is on PATH (`adb version`). If not, point the user at
   Android Studio's Platform Tools.
4. Ask: Anthropic API key or Ollama? Most users want Anthropic for
   quality; Ollama if cost-sensitive or curious. Help them set up
   `.env` from `.env.example` accordingly. See the README "Backends"
   section for the toggle.

### Phase 2 — Emulator + Hinge

1. The user needs an Android emulator running. Pixel 10 (1080×2424) is
   the calibrated default; other devices will need recalibration.
2. The user has to sideload the Hinge APK themselves — direct them to
   their preferred APK source. Do not link to or recommend specific
   pirate APK sites.
3. After install, the user signs in (throwaway account, see Hard
   Constraints) and navigates to the Discover tab.
4. **Strongly recommend Hinge+.** Without it the user is capped at
   ~8–10 likes/day on the free tier, which makes this tool pointless.
   With it, the bot effectively becomes the subscription's labor — the
   user gets full daily-like-allotment value without ever opening
   Hinge. Frame it that way, not as an upsell.
5. Run `adb devices` to confirm the emulator is visible. Troubleshoot
   if not (most common issue: emulator not started, or USB debugging
   off on a physical device).

### Phase 3 — Calibration

The shipped `COORDS` in `config.py` are placeholders. They will be
wrong for the user's emulator.

1. With Hinge open on the Discover tab in the emulator, run
   `python calibrate.py`. It saves `calibrate.png` to the repo root.
2. Open `calibrate.png` in any image viewer that shows cursor pixel
   coordinates (Paint on Windows, Preview's "Show Inspector" on Mac,
   any image-coord browser extension).
3. The user reads off pixel coordinates for: skip button, heart on
   photo 1, send-like button, comment input, scroll start/end. Help
   them update `config.COORDS` in `config.py` with the values.
4. **Verify by inspection** — show the user the diff of `config.py`
   before/after, and have them sanity-check that the coords look like
   what they read off the screenshot.

If the user wants to use `--set-filters`, `--location`, or
`--rotate`, they also need to run `calibrate_filters.py` /
`calibrate_matches.py` and hand-edit `location_coords.json` (no
interactive helper exists for the location picker yet).

### Phase 4 — Write a mode

1. Show the user `modes/example_lenient.py` and `modes/example_strict.py`
   as the two contrasting starting points.
2. Ask what kind of rubric they want — generous (mostly likes,
   minimal filtering) or selective (defaults to skip, only likes on
   strong signals)?
3. Copy the closer example to `modes/<user_chosen_name>.py`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TerraByte-Dev/hinge-auto](https://github.com/TerraByte-Dev/hinge-auto) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
