---
trigger: always_on
description: - Full local preflight (matches CI):
---

# Copilot instructions for `ha-govee-led-ble`

## Build, lint, and test commands

- Full local preflight (matches CI):  
  `bash scripts/check.sh`
- Completion gate: after making changes, `bash scripts/check.sh` must pass; if it fails, fix the issue and rerun until it passes, then capture any durable repo-specific lesson in these instructions.
- Run a single test:  
  `uv run pytest tests/test_protocol.py::test_parse -q`

`check.sh` owns the stage list and the exact flags. Do not restate them here: a second copy drifts silently, and the version that lived here had already lost the `--no-sync` the script uses.

## High-level architecture

- This is a Home Assistant custom integration (`domain: ha_govee_led_ble`) for local BLE control of supported Govee models (currently H617A and H6199).
- `config_flow.py` handles discovery/manual setup, infers model from BLE local name, and creates config entries keyed by device address.
- `__init__.py` creates one `GoveeBLECoordinator` per config entry, performs first refresh, removes legacy entities, and forwards setup to the platforms listed in its `PLATFORMS` constant.
- The coordinator is split across `coordinator*.py`: BLE connect/reconnect lifecycle, notification subscription, keep-alive/state queries, optimistic state fields, and bounded raw packet logging for diagnostics.
- Kaitai schemas own wire structure. Committed modules in `generated_protocol/` are generated from them; handwritten protocol code retains only semantic transforms, checksums and transport framing.
- `light.py` is the primary control surface, with the custom services in `light_services.py`.
- `h6199_controls.py` contains shared advanced control entities for Number/Select/Switch; `number.py`, `select.py`, and `switch.py` are thin entry-point wrappers.
- `scenes.py` loads the committed per-model scene snapshots used by light effect selection.

Name a module here only when something else in this file depends on knowing it exists. A full inventory rots: the last one still listed four platforms after there were seven.

## Key repository conventions

- Model capabilities are declared in `const.py` via `ModelProfile` fields such as `supports_scenes`, `supports_scene_speed`, `supports_video_mode`, `supports_white_balance`, `supports_blank_screen`, `static_readback_echoes_color`, and segment fields. New model behaviour should be wired through a profile field first, then entity setup. `supports_segments` and `supports_music_mode` are derived properties, so check before trying to set them.
- Prefer root-cause refactoring over band-aid fixes; when behavior crosses layers, update shared paths instead of patching a single call site.
- Treat changes holistically across capabilities, protocol encode/decode, coordinator state handling, entity/service wiring, diagnostics, and tests so behavior stays consistent.
- Advanced entities are capability-gated at setup time (see `h6199_controls.py`), so unsupported controls are not created for a model.
- Do not add wire offsets, literals or enums to entity/coordinator code. Put structure in Kaitai and keep only semantic transforms, checksums and transport framing handwritten.
- State writes are optimistic but guarded:
  - `light.py` uses `_rollback()` snapshots plus `_refresh_with_retry()` verification for state-readable models.
  - `h6199_controls.py` uses `_set_with_rollback()` around reapply callbacks.
- Effect names are normalized (`_normalize_effect_name`) before lookup/comparison; preserve this normalization path when adding new effects/services.
- `scripts/check.sh` is treated as the authoritative local validation flow and should stay aligned with `.github/workflows/validate.yml`.

## Protocol source of truth

- Captures are ground truth.
- `tools/ble/kaitai/*.ksy` is the only wire-structure source. Do not restate offsets,
  literals or enums elsewhere.
- Unknown attributes follow official Kaitai style and omit `id`. `reserved` means known
  unused. Unparsed transport chunks are not protocol unknowns.
- `govee_shared.ksy` contains structures independently exercised through both models;
  model-specific roots remain separate.
- Every fixture has machine-readable provenance and a committed SHA-256. Cross-fixture
  claims live in `spec/_aggregates.yaml`; the runner hard-fails missing, skipped or stale
  coverage.
- `scripts/generate-kaitai.sh` uses the pinned Kaitai compiler from `mise.toml`. Java is
  an unpinned development/CI runtime only.
- Generated Python in `custom_components/ha_govee_led_ble/generated_protocol/` is
  committed and never edited manually.
- After changing KSY or fixtures, run `bash scripts/check-kaitai.sh`.

## Captures: one capture, one light

A phone stays paired with every Govee device in the house, so an HCI capture is not evidence about a model until it is attributed. Two mechanisms enforce that, and both exist because the failure they catch reads as absence rather than as an error.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [teh-hippo/ha-govee-led-ble](https://github.com/teh-hippo/ha-govee-led-ble) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
