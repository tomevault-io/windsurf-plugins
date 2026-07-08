---
trigger: always_on
description: Python bot that auto-plays the Dota 2 "Bootbreaker" arcade minigame (a
---

# Bootbreaker Autoplayer

Python bot that auto-plays the Dota 2 "Bootbreaker" arcade minigame (a
Breakout/brick-breaker variant: bounce a flying "boot" ball with a cart/paddle).
**Primary goal: keep the ball alive as long as possible.** Survival = winning;
no other objectives matter.

## Platform & tooling constraints (hard requirements from the user)

- **Windows only.**
- **Use `uv` for everything.** Deps live in `pyproject.toml`.
- **Do NOT pin versions** — always use latest.
- **Keep it lean** — only add a dependency if actually needed.
- Run: `uv run python -m bootbreaker` (add `--debug` or `--recalibrate`).
- Tests: `uv run pytest` (currently ~43 tests, all passing).

## Scope: fully autonomous (auto-throw + tracking)

The bot both **launches/aims the ball and tracks it.** Three states (`bot.py`):
- **AIM** — the on-screen pre-throw prompt (the ␣ key icon) is visible and no
  ball is in play -> returns `"launch"`. Main's `_launch()` locks the cart (tap
  Space), then **steers the aim to vertical with A/D in a closed loop** and
  throws (tap Space). The aim is NOT an auto-sweep we wait on — it's driven by
  A/D, so `_launch` pulses A/D (`_AIM_STEP_HOLD=0.05s` per pulse), re-reads
  `detect_aim_angle`, and stops when within `_AIM_TOLERANCE=5°` of straight-up
  (deadband can't be 0 or it oscillates forever) or after `_AIM_MAX_ADJUST=40`
  pulses. We don't know which of A/D rotates which way, so the loop **self-
  corrects**: if a pulse doesn't reduce the tilt, it flips the key. `_launch`
  logs the last aim samples for tuning. **The prompt is the ONLY launch trigger.**
- **PLAYING** — a ball is in play -> track and steer the cart to intercept.
- **WAIT** — no ball and no prompt (a brief transition): hold (steer to the last
  target for `chase_gap` frames first, in case of a blind-spot blip).

Launch used to fire from a ball-absence heuristic (miss counting), which threw
at random moments. It's now driven by `detect_prethrow` (template-matching the
grey ␣ key icon), so it only throws when the game is actually asking for it.
If the near-vertical aim reading proves unreliable, the fallback is to simplify
`_launch` to lock+throw immediately (tracking handles angled launches).

## Controls (in-game)

- `A` = move cart left, `D` = move cart right (bot presses these).
- `Space` = lock cart + throw — **pressed by the bot** in `_launch()`.
- `F8` = global hotkey to start/pause the bot. **Starts PAUSED.** User opens
  Dota, presses F8, which triggers play-region auto-detection on first run.

## Dependencies

Runtime: `mss` (screen capture), `opencv-python` + `numpy` (HSV color
detection), `pydirectinput` (scancode key output — works in games where normal
key injection doesn't), `keyboard` (global F8 hotkey). Dev: `pytest`.
Build: `hatchling`, `packages = ["bootbreaker"]`.

## Config (per-machine, git-ignored)

- `config.json` holds the play region `{left, top, width, height}`. It is
  **git-ignored** — different on every machine (different resolution/DPI).
- `config.example.json` is the committed template.
- `bootbreaker/config.py` loads/saves `config.json` (`DEFAULT_CONFIG_PATH`).
- First run (or `--recalibrate`) auto-detects the region via
  `detect.detect_play_region` and saves it.

## Architecture / files

- `bootbreaker/main.py` — CLI (`--debug`, `--recalibrate`), F8 pause toggle,
  and the capture→decide→act loop. Each frame: `bot.step()` returns
  "launch"/"left"/"right"/"hold"; `"launch"` runs `_launch()` (lock+aim+throw)
  then `bot.prime_playing()`, otherwise the loop holds `A`/`D` or releases keys.
  `--debug` shows a live always-on-top overlay window (ball=cyan, cart=red line
  + deadzone band, target=green line, HUD with state/action/coords) refreshed
  every 3rd frame, and prints an fps readout. During `_launch` (debug) the same
  window shows the **aim overlay**: detected aim dots (yellow), the fitted aim
  line (magenta), the search band (grey), and a true-vertical reference at the
  cart (green) — magenta not parallel to green means the throw isn't straight up.
- `bootbreaker/capture.py` — `grab(region, sct)`, `grab_fullscreen`,
  `calibrate(config_path, grabber)` (auto-detects + saves; raises if not found).
- `bootbreaker/detect.py` — **the most-iterated, most-fragile file.** HSV
  color detection of region, ball, cart, aim angle. See notes below.
- `bootbreaker/strategy.py` — pure functions: `decide_move(target_x, cart_x,
  deadzone)` → "left"/"right"/None (bang-bang w/ deadzone);
  `predict_intercept_x(ball_x, ball_y, vx, vy, paddle_y, width)` reflects the
  ball's path off side walls (triangle wave) to predict the landing x;
  `_reflect`.
- `bootbreaker/bot.py` — `Bot`, the AIM/PLAYING state machine + tracker. See
  notes below.
- `bootbreaker/input.py` — `Controller(backend=pydirectinput)`:
  `hold(key)` / `release_all()` / `tap_space()`, tracks held keys. `tap_space`
  holds Space down for `_TAP_HOLD=0.06s` (down→sleep→up), NOT `press()` — see
  the PAUSE/tap gotcha below.
- Tests in `tests/`; screenshots the detection is tuned against in `docs/`.

## Detection notes (`detect.py`) — hard-won, don't regress

- **Ball = flying boot.** It has a **cyan glow ring** + a **brown boot body**.
  `detect_ball` finds cyan blobs (HSV `_CYAN_LO`–`(110,255,255)`,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jackblk/dota-bootbreaker-auto](https://github.com/jackblk/dota-bootbreaker-auto) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
