---
trigger: always_on
description: generates from it, so it has to be right in the `.so` files.
---

# CLAUDE.md

SSTVAE: image transmission over HF radio by sending convolutional
autoencoder latents as analog values on OFDM carriers (RADE-style).
See README.md for the waveform table and usage; the approved design
rationale lives in the plan history.

## Commands

- Run tests: `pytest` (fast, ~10 s; includes full modem end-to-end tests)
- Slow gate: `pytest -m slow` (~2 min) — the listener state machine and
  the app's transmit→receive loopback. Run it after touching `sstvae/rx/`.
- Native port: `tools/build_native.sh --test` (builds `native/`, runs
  `ctest` and `pytest --native`). See "The native port" below.
- Run the app: `tools/build_native.sh` then `native/build/sstvae-gui`
  (the app is C++; there is no Python GUI any more)
- Smoke-train: `python scripts/train.py --smoke --out /tmp/smoke`
- Full pipeline check: `sstvae_encode.py` → `sstvae_simulate.py` → `sstvae_decode.py`

## Testing the live paths without hardware

Both of these exercise the *real* code paths, which is the point — the
audio and rig bugs found so far were all invisible to unit tests.

- **Rig control:** set the app's rig model to **1**, Hamlib's dummy rig,
  and PTT, frequency readback and the whole `RigController` threading
  model can be driven for real without a radio attached. Model **2**
  (NET rigctl) against a `rigctld -m 1 -t <port>` exercises the shared-
  radio path the same way.
- **Audio loopback:** a null sink plus a *remapped* monitor, because Qt
  does not enumerate monitor sources:

  ```sh
  pactl load-module module-null-sink sink_name=null-sink
  pactl load-module module-remap-source source_name=sstvae_loop \
      master=null-sink.monitor channels=1 \
      source_properties=device.description=SSTVAE-Loopback
  ```

  Then play into `null-sink` and capture `SSTVAE-Loopback`. Unload the
  modules by index (`pactl unload-module N`) afterwards. **Pre-resample
  the file to the sink's rate** — `pw-play` converting 44.1k→48k on the
  fly cost ~4 dB of apparent SNR and sent me chasing a phantom.
- **Anything Qt with an event loop: run it under `timeout`.** A headless
  `QApplication` with `app.quit()` called from a worker thread has hung
  this project's runs; `timeout 120 uv run python ...` makes that
  self-limiting. Do not put event-loop tests in the pytest suites.

## Architecture

- `sstvae/config.py` — every constant shared between modem, channel sim,
  and training. **All waveform/latent numbers must agree through this
  module**. One carrier (`BEACON_CARRIER`) is permanently reserved for
  the beacon side-channel, so `LATENTS_PER_FRAME` (23-carrier capacity)
  no longer evenly divides `GROUP_LATENTS` (132ch model contract);
  `FRAMES_PER_GROUP` is pinned to the *pre-beacon* 24-carrier capacity
  instead (so this is a capacity trade, not a time trade — mode
  durations are unchanged), and the `DROPPED_LATENTS_PER_GROUP`
  remainder per group (~4.2%) is a permanent erasure, never transmitted.
- `sstvae/modem/` — NumPy DSP, no torch:
  - `ofdm.py` DFT-matrix mod/demod (24 carriers × 50 Hz at 950–2100 Hz;
    carriers on integer multiples of 50 Hz so the CP is truly cyclic).
    **The pilot is a minimized-crest-factor phase set, 0.99 dB envelope
    PAPR (`PROTOCOL_VERSION` 3, 2026-08-14)** — see "The pilot is three
    things at once" below. It replaced a frozen random QPSK draw at
    7.9 dB, and `CLIP_HEADROOM_DB` moved 0.5 → 0.0 in the same change.
  - `sync.py` preamble detect (lag-160 autocorrelation over a
    480-sample window, energy-floored metric), fractional + integer-bin
    CFO, template timing.
    **The preamble is four repeats, not two, and the correlation window
    is what the length buys** (2026-08-04, `PROTOCOL_VERSION` 2). At two
    repeats the metric's noise floor sat *inside* the threshold: on AWGN
    the median 5-second maximum measured 0.461 against a threshold of
    0.50, i.e. 0.47 crossings per second of noise, and the only gate
    behind it is the header — which admits 3 of 4096 Golay codewords
    (7.2e-4, measured) and **cannot be tightened without more header
    bits**, 3 valid messages in a 12-bit payload being its floor. That
    product is the false lock every few hours a live receiver sees on a
    quiet band. Four repeats at threshold 0.42 give **no crossing at all
    in 3000 s** (24M positions, peak 0.358) *and* mode A acquisition at
    −2 dB of 0.93 against 0.40 — both directions at once, which the old
    preamble could not do: raising *its* threshold to 0.64 cleared the
    false alarms too and cost 0 dB acquisition (0.82 → 0.53) to do it.
    Costs 40 ms on a 32–95 s transmission. The trap is that
    `PREAMBLE_SAMPLES` and `PREAMBLE_CORR_WINDOW` are **one change, not
    two**: a longer preamble read through the old one-symbol window has
    exactly the old noise floor and the old false-alarm rate, with every
    other test still passing, which is what `tests/test_preamble.py`
    exists to catch (through the metric's output length, since a stale
    kernel is invisible in its values). Measured on AWGN — what the
    field false locks actually trigger on is unknown, and an MPP number
    here would be reporting a ±4-sample timing criterion rather than
    acquisition.
    `acquire_blind()` is a separate, preamble-free path: matched-filters

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [arodland/SSTVAE](https://github.com/arodland/SSTVAE) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
