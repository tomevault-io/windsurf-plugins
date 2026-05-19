---
trigger: always_on
description: Real-time collaborative visual art synthesizer for live performance. Dual-source video mixer with a GPU-accelerated effect chain, LFO modulation, audio reactivity, MIDI/OSC control, and a REST API + React web UI. Target audience: VJs, live coders, audiovisual artists.
---

# Video Synth — Claude Code Context

## Project Identity

Real-time collaborative visual art synthesizer for live performance. Dual-source video mixer with a GPU-accelerated effect chain, LFO modulation, audio reactivity, MIDI/OSC control, and a REST API + React web UI. Target audience: VJs, live coders, audiovisual artists.

## Key Documents

- **[docs/roadmap.md](docs/roadmap.md)** — Full phased development roadmap (phases 1–4, ~25 items across engine, UI, AI, packaging, DX)
- **[docs/api.md](docs/api.md)** — REST API reference and integration examples
- **[docs/parameters-reference.md](docs/parameters-reference.md)** — All animation/effect parameters with min/max/default values

## Running the App

```bash
# Desktop GUI (default — run from project root)
PYTHONPATH=src:src/video_synth python -m video_synth

# Headless API server (web UI only, no Qt window)
PYTHONPATH=src:src/video_synth python -m video_synth --headless --api --api-host 0.0.0.0

# With virtual camera disabled (required in Docker / CI)
PYTHONPATH=src:src/video_synth python -m video_synth --headless --api --no-virtualcam

# Full Docker stack (video synth + Ollama agent)
docker compose up --build
```

Service URLs when running:

- Web UI (React): `http://localhost:8000/ui/`
- API docs (Swagger): `http://localhost:8000/docs`
- AI agent chat: `http://localhost:8001/`
- MJPEG stream: `http://localhost:8000/stream`

## Repo Layout

```text
src/video_synth/     Python package (src/ layout — PYTHONPATH=src:src/video_synth)
agent/               Ollama LLM agent service (separate Docker container)
build/               PyInstaller spec, hooks, build scripts
docs/                All documentation (MkDocs source + reference files)
save/                Runtime YAML state (patches, MIDI maps, autosave)
scripts/             Developer tooling (write_patches.py, etc.)
shaders/             GLSL shader files
tests/               pytest suite
web/                 React + Vite web UI (npm run build → web/dist/)
```

## Architecture

```text
src/video_synth/__main__.py   CLI entry point, main video loop thread
  ├── Mixer                   dual-source blend (ALPHA, LUMA_KEY, CHROMA_KEY)
  ├── EffectManager           chains all effects for one source (src_1, src_2, post)
  │     └── effects/          Color, Pixels, Warp, Glitch, Feedback, Shapes, ...
  ├── animations/             procedural frame generators (Metaballs, Plasma, DLA, ...)
  ├── LFO / OscBank           real-time parameter modulation
  ├── AudioReactive           FFT bands → parameter modulation
  ├── APIServer               FastAPI + uvicorn on port 8000
  ├── MidiMapper              generic MIDI learn/map system
  ├── OSCController           OSC server (TouchOSC, SuperCollider, etc.)
  ├── OBSController           OBS WebSocket integration
  └── SaveController          YAML patch load/save
```

Frame pipeline per loop iteration:

1. Each EffectManager calls the active animation's `get_frame()`
2. Effects are applied in chain order to the frame
3. Mixer blends src_1 and src_2 outputs using post_effects
4. Frame is sent to: GUI, virtual cam, FFmpeg, APIServer (for /stream and /snapshot)

## Key Conventions

### Adding an Animation

- File goes in `src/video_synth/animations/`
- Class extends `Animation` from `animations.base`
- Constructor registers all params via `params.new(name, min=, max=, default=, subgroup=ClassName, group=group)`
- Must implement `get_frame(self, frame: np.ndarray) -> np.ndarray`
- Register in `animations/enums.py` and `effects_manager.py`
- See `animations/metaballs.py` for the canonical pattern

### Adding an Effect

- File goes in `src/video_synth/effects/`
- Class extends `EffectBase` from `effects.base`
- Same param registration pattern as animations
- Method name follows the `do_xxx` pattern (e.g., `do_color()`, `do_warp()`)
- Register in `effects_manager.py`
- See `effects/color.py` for the canonical pattern

### Parameter System

- `params.new()` returns a `Param` object; access its value via `.value`
- `Widget.DROPDOWN` type enables enum selectors in both the Qt and web UIs
- All params are automatically exposed via `/params` REST endpoint
- Min/max bounds are enforced by the API; the UI uses them for slider range
- LFOs can be attached to any param via `/lfo/{param_name}` API

### Patch System

- Patches live in `save/saved_values.yaml` as a list under `entries:`
- `scripts/write_patches.py` is the scripting tool for generating patch entries
- LFO shape constants: NONE=0, SINE=1, SQUARE=2, TRIANGLE=3, SAWTOOTH=4, PERLIN=5
- `save/autosave.yaml` is written on exit and loaded on next start
- `save/midi_mappings.yaml` persists MIDI controller learn mappings

## Docker Stack

- **video_synth** (port 8000): headless API mode, Mesa software GL, Xvfb virtual display
- **ollama** (port 11434): Ollama inference server, model weights in `ollama_data` volume
- **agent** (port 8001): FastAPI + OpenAI-compatible client → Ollama, web chat UI
- Model: `llama3.2:3b` by default; change `OLLAMA_MODEL` in `docker-compose.yml`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Khendi1/PVS](https://github.com/Khendi1/PVS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
