---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running

```bash
python -m c64cast -u u64://192.168.2.64 -d 0
# or with a config file (overrides defaults; CLI flags still win):
python -m c64cast --config c64cast.toml
# or quick playback — one scene per positional MEDIA arg, no config file:
scripts/c64cast.sh -u tr:// clip.mp4 tune.sid assets/pictures/
```

[scripts/c64cast.sh](scripts/c64cast.sh) is the equivalent launcher for shells where direnv hasn't activated `.venv` — see ["Running from a checkout"](CONTRIBUTING.md#running-from-a-checkout).

**Connection target (`-u/--url`).** One scheme-aware string selects the backend and its endpoint: `u64://HOST` or `http(s)://HOST` (Ultimate 64), `tr://` (TeensyROM+ USB serial, auto-detected), `tr:///dev/cu.usbmodemXYZ` / `tr://COM3` (explicit serial), `tr://HOST[:PORT]` (TeensyROM+ TCP, default 2112); per-link knobs ride as `?query` params, `$C64CAST_URL` is the env fallback. Details: [c64cast/app/connect.py](c64cast/app/connect.py)'s docstring.

**Quick playback** (positional `MEDIA` args, mutually exclusive with `--config`) builds an in-memory-only config — one scene per argument by extension/URL, no loop unless `--loop`. Details: [c64cast/app/quickcast.py](c64cast/app/quickcast.py)'s docstring; URL resolution is shared with config-driven video scenes ([`scene_factory.py`](docs/architecture/config.md#scene_factorypy)).

**Audio is on by default**; `--no-audio` mutes. U64 video audio uses the Ultimate Audio FPGA PCM sampler when available; `[audio].backend = "dac"` forces the 4-bit `$D418` DAC (the only path on TeensyROM and for mic/webcam audio). The sampler's effective clock ships as 6160000 Hz, not the nominal 6.25 MHz — why, and how to re-measure: [audio.md → sampler](docs/architecture/audio.md#samplerpy--ultimateaudiosampler-u64-ultimate-audio-fpga-pcm).

Flag groups (`-h` shows them grouped): `connection`, `quick playback`, `video input`, `audio`, `vision input`, `playlist`, `introspection`, `debug`. Notable: `--config`, `-v` / `-vv`, `--log-file PATH` (each scene activation logs a `SCENE_CONFIG_JSON` snapshot safe for a public video description — [`recording_metadata.py`](docs/architecture/config.md#recording_metadatapy--per-scene-scene_config_json-logging)).

The DMA password (if the U64 has one set) is supplied via `C64CAST_DMA_PASSWORD` env var or `[ultimate64] dma_password` in the config — **no CLI flag**, so secrets don't leak into shell history or `ps` output. The env var wins when both are set.

**Firmware prerequisites:** three U64 switches in two menus (Ultimate DMA Service, Web Remote Control Service, Command Interface), plus **Bus Operation Mode = `Writes`** when a TeensyROM+ sits in an Ultimate's cartridge port — walkthrough in [docs/guide/04-setting-up.md](docs/guide/04-setting-up.md), symptoms in [docs/caveats.md](docs/caveats.md). The REU and the sampler are *not* switches — `hw_provision` enables them live+volatile per run.

Hard deps: `opencv-python`, `numpy`, `requests`, `py65`; optional extras are grouped in [pyproject.toml](pyproject.toml) (`video` = PyAV, `yt` = yt-dlp, `wizard` = `--init`, …, `all`), and dev tools are a PEP 735 `dev` group installed by default.

**Setup is `uv sync --all-extras` — never `uv pip` and never raw `pip`** (the mise/`UV_PYTHON` trap: ["Development setup"](CONTRIBUTING.md#development-setup)). The `make` targets route through `uv run`, so they hit the synced project env from any shell; `make doctor` is the fast offline self-check (environment-probe rationale on the probe docstrings in [c64cast/app/doctor.py](c64cast/app/doctor.py)); type-checking is `pyright` basic tree-wide + `mypy --strict` on the state-bearing modules — ["The pre-PR gate"](CONTRIBUTING.md#the-pre-pr-gate).

Target hardware: an [Ultimate 64](https://ultimate64.com/) on the LAN. Writes go over the **Ultimate DMA Service** (TCP port 64, persistent socket); reads, reset, run_prg, and probe go over REST. SID playback DMAs the payload + a tiny 6502 player into C64 RAM and kicks a `SYS` stub through `run_prg` — the firmware's `runners:sidplay` endpoint is deliberately avoided because it hijacks HDMI. See [api.run_sid_player](c64cast/hw/api.py) and [docs/caveats.md](docs/caveats.md).

## Configuration

TOML file (`--config PATH` wins; else `./c64cast.toml` if present; else built-in defaults). See [c64cast.example.toml](c64cast/examples/c64cast.example.toml) for an annotated reference.

**Precedence (single system):** dataclass defaults → **machine settings** → project/per-system TOML → CLI → env (`C64CAST_DMA_PASSWORD`). **Ensemble (per system):** defaults → machine settings → per-system TOML → master cascade (fills only fields still at the machine-overlaid baseline) → CLI/env. Every layer above the defaults overrides the ones below it.

Every CLI flag has `default=None`; `config.merge_cli()` only overwrites a config field when the CLI value is non-None.

Three mechanisms are detailed in [docs/architecture/config.md](docs/architecture/config.md):


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kfox/c64cast](https://github.com/kfox/c64cast) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
