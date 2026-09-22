---
trigger: always_on
description: Python port of [ot-tools-io](https://gitlab.com/ot-tools/ot-tools-io/) (Rust). Reads, writes, and modifies Elektron Octatrack project files (`project.work`, `bank01..16.work`, `markers.work`).
---

# octapy — agent guide

Python port of [ot-tools-io](https://gitlab.com/ot-tools/ot-tools-io/) (Rust). Reads, writes, and modifies Elektron Octatrack project files (`project.work`, `bank01..16.work`, `markers.work`).

## Code philosophy

- **No backwards compatibility shims.** When refactoring, delete the old code. No `_old` / `_legacy` suffixes, no `# deprecated` markers, no re-exports of moved modules. The recent history has examples — `RenderSettings`, `TEMPLATE_DEFAULT_*` constants, machine-named `flex_track()` / `static_track()` accessors all got deleted outright rather than aliased.
- **Match the binary spec.** When in doubt about an offset, enum value, default, or struct layout, verify against `/Users/jhw/work/ot-tools-io/ot-tools-io/src/`. That Rust crate is the authoritative reference. The bundled `octapy/templates/project-template-1.40B.zip` is the on-device factory template captured byte-for-byte.
- **Be honest about what each piece is for.** `_io/` is the binary layer, `api/` is the high-level surface. Don't leak `_io` constants into user-facing code unless there's a real reason. Don't add a class when a method on `Project` will do (see `configure_recorder_buffer` — it replaced a god-shaped `RenderSettings` knob).
- **Simple and focused.** Implement what the task needs; don't speculate. Three similar lines beats a premature abstraction.

## Architecture

- **Core library:** `octapy/api/` — `enums.py`, `settings.py`, `sample_pool.py`, `slot_manager.py`, `utils.py`, plus the `core/` subtree (`project.py`, `bank.py`, `pattern.py`, `part.py`, `scene.py`, `_page.py`, `_lfo_plock.py`, `_trig.py`, `audio/`, `midi/`).
- **Low-level binary I/O:** `octapy/_io/` — `BankFile`, `MarkersFile`, `ProjectFile` (text/INI). Each wraps a `bytearray` and exposes typed accessors via offset `IntEnum`s.
- **Tools:** `tools/sync.py` (unified push/clean/status for projects + samples). The one-off Erica Pico downloader lives separately.
- **Templates:** `octapy/templates/project-template-1.40B.zip` — the factory project files bundled into the wheel.

### Hierarchy reflected in the API

```
Project → Bank → Pattern → audio_track/midi_track → step
                ↘ Part → audio_track/midi_track + scene → (track scene locks + XLV)
```

This mirrors the Octatrack's own UI hierarchy. Don't invent intermediate concepts.

### Page accessors (`api/core/_page.py`)

`AudioPartTrack` and `AudioSceneTrack` expose machine-aware page accessors:

```python
track.src.pitch        # FLEX/STATIC: pitch; THRU: in_ab; PICKUP: pitch
track.setup.loop       # FLEX/STATIC: loop mode; PICKUP: timestretch
track.amp.attack       # AMP attack
track.fx1.base         # FX-type-aware (FILTER: base, EQ: freq1, …)
```

The accessor uses the current `machine_type` (or `fx1_type` / `fx2_type`) to look up parameter names from `SRC_PARAM_NAMES` / `SRC_SETUP_PARAM_NAMES` / `FX_PARAM_NAMES`. Adding a new machine or FX type means extending the appropriate dict in `_page.py` and the offset enum in `_io/bank.py`.

### Step accessors

Per-step state lives in three places:

- **Trig flags** — `step.active`, `step.trigless`, `step.swing`, `step.slide` (audio); `swing` only on MIDI. Backed by 8-byte trig masks on the parent track, synced via a callback.
- **Condition** — `step.condition` (TrigCondition enum) or `step.probability` (float, quantizes to nearest valid percent). Stored in a 2-byte condition field per step.
- **P-locks** — 32 bytes per step. `step.volume`, `step.pitch`, `step.start`, etc. on audio; `step.note`, `step.velocity`, `step.cc(n).value`, `step.lfo(n).speed/.depth` on both audio and MIDI. `None` = no lock.

`step.lfo(n)` and `step.cc(n)` return small accessor objects (`LfoPlock`, the CC accessor) for symmetry with `track.lfo(n)` — the shape is uniform across part-tracks and steps.

### Defaults — octapy baseline vs OT factory

Every construction path lands on the **octapy baseline**: `length=127`, `length_mode=TIME`, `loop=OFF` on the SRC page, and the recorder's `rlen=16`, `qrec=PLEN`, sources OFF. These differ from the OT's factory defaults in a small number of fields, chosen for programmatic one-shot sample workflows.

- `_io/bank.py` defines two parallel sets of constants: `OT_FACTORY_*` (the device's factory values, kept as a reference) and `OCTAPY_DEFAULT_*` (what octapy actually writes).
- `BankFile.new()` applies `OCTAPY_DEFAULT_*` across all 64 part-tracks when loading the template.
- `AudioPartTrack._apply_defaults` writes `OCTAPY_DEFAULT_*` for the FLEX machine on standalone construction.
- `AudioRecorderSetup.__init__` writes `OCTAPY_DEFAULT_RECORDER_SETUP`.
- Escape hatch: `track.reset_to_factory_defaults()` flips back to `OT_FACTORY_*` and resets the recorder too. Symmetric with `track.apply_recommended_defaults()`.

If you're tempted to make a path use `OT_FACTORY_*` directly, stop — every construction path should produce identical state. The asymmetry is a load-bearing invariant of the test suite.

### Settings (grouped)

`Settings` exposes grouped accessors that mirror the OT's PROJECT menu:

- Top-level scalars: `tempo`, `master_track`, `write_protected`, `pattern_tempo_enabled`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jhw/octapy](https://github.com/jhw/octapy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
