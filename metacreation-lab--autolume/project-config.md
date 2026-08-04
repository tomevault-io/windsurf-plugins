---
trigger: always_on
description: Guidance for AI coding assistants working on Autolume. Humans should read [CONTRIBUTING.md](CONTRIBUTING.md) instead — most of this file is the same information distilled into a denser form.
---

# AGENTS.md

Guidance for AI coding assistants working on Autolume. Humans should read [CONTRIBUTING.md](CONTRIBUTING.md) instead — most of this file is the same information distilled into a denser form.

## Project snapshot

Autolume is a no-coding generative AI system for real-time visual performances using StyleGAN. Target users are artists who train, craft, and perform with their own generative models. Built on Python 3.12 + PyTorch 2.8 (CUDA 12.8), with an imgui-based desktop UI. Windows is the primary platform; Linux (Ubuntu 24.04) is supported.

## Commands

| Command | Purpose |
|---------|---------|
| `uv sync` | Install or update the locked dependency set |
| `uv run main.py` | Launch the application |
| `uv run zensical build` | Build the documentation site to `site/` |
| `uv run zensical serve` | Serve docs locally at http://127.0.0.1:8000 |
| `uv run release.py` | Build the PyInstaller release (cross-platform): `dist/Autolume/` on Windows/Linux, `dist/Autolume.app` on macOS. Run it on the target OS — PyInstaller cannot cross-compile |

Initial setup also requires CUDA 12.8, MSVC build tools (Windows), `ffmpeg` (Linux), pre-trained Real-ESRGAN/face-parsing models, and an FFmpeg binary on Windows. Full details in the [README](README.md#development-instructions).

## Architecture

```
autolume/
  main.py                     # Application entry point
  pyproject.toml              # Dependencies, Python pin (3.12), CUDA torch index
  release.py                  # Cross-platform release script (drives PyInstaller)

  architectures/              # Generator/discriminator network definitions
  audio/                      # Audio capture and analysis (sounddevice, numpy)
  bending/                    # Network bending operators applied at inference
  dnnlib/                     # Vendored NVIDIA dnnlib (StyleGAN2/3)
  ganspace/                   # GANSpace PCA latent direction discovery
  metrics/                    # FID, KID, and related quality metrics
  modules/                    # UI modules and live-performance widgets
  widgets/                    # Reusable imgui widgets
  projection/                 # Image-to-latent projection
  sr_models/                  # Super-resolution model weights (gitignored)
  super_res/                  # Real-ESRGAN super-resolution wrappers
  torch_utils/                # Vendored NVIDIA torch utilities (custom ops, persistence)
  training/                   # Model training and distillation
  utils/                      # Shared helpers (datasets, GUI, paths, version)
  assets/                     # Bundled images and icons

  docs/                       # User documentation (zensical, versioned with mike)
  .github/                    # Issue/PR templates, workflows
```

## Key Files

- [main.py](main.py) — entry point; wires the UI and rendering pipeline.
- [pyproject.toml](pyproject.toml) — pinned dependencies; `torch==2.8.0+cu128` is a hard requirement, do not relax it.
- [release.py](release.py) — cross-platform release script (`uv run release.py`); detects the host OS, resolves package locations via `importlib`, and assembles the PyInstaller `--add-binary`/`--add-data` flags per platform, then copies `sr_models/` and creates runtime dirs. **This is where to add new runtime files** — the auto-generated `Autolume.spec` is gitignored and rebuilt every release. Windows/Linux bundle the runtime JIT toolchain (torch headers/libs, ninja); macOS skips it (ops fall back to reference PyTorch on MPS) and produces `Autolume.app`. Requires a repo-root `.env` with the crash report endpoint values (baked in at build time) or it fails fast — see `.env.example` and [tools/crash_endpoint/README.md](tools/crash_endpoint/README.md); `--disable-crash-reporting` builds without one (for forks).
- [utils/user_data.py](utils/user_data.py) — user preferences and writable data paths. Preferences (data root, UI font size, …) persist to a JSON file at `~/.config/autolume/config.json` (`XDG_CONFIG_HOME` honored); the Settings modal ([modules/settings.py](modules/settings.py)) is their UI. **Add new user-facing preferences here** (a `pref()`/`set_pref()` accessor pair following the existing ones), not as ad-hoc files.
- [utils/gui_utils/dpi.py](utils/gui_utils/dpi.py) — all display-scale math (monitor DPI scale, font atlas raster scale, 1x text sharpening). The UI is sized in DPI-independent units so it keeps the same physical size and layout on every monitor and platform; start here for any scaling/blurriness issue.
- [.github/workflows/docs.yml](.github/workflows/docs.yml) — only CI workflow; publishes versioned docs.
- [zensical.toml](zensical.toml) — docs site config (Material theme variant).

## Code Style

- Project predates a formal style enforcer; match the surrounding code.
- Wrap `Path` / `WindowsPath` objects with `str()` before passing them anywhere that needs JSON-serializable values (training kwargs, OSC payloads, preset files). This is a recurring source of bugs.
- Imports follow standard Python order; vendored NVIDIA modules (`dnnlib`, `torch_utils`) keep their upstream style — do not reformat them.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Metacreation-Lab/autolume](https://github.com/Metacreation-Lab/autolume) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
