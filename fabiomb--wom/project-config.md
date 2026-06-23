---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

WOM is a turn-based 2D military strategy game. Supports **free-for-all de 2 a 4 jugadores** (`MAX_PLAYERS = 4` en `wom/core/worldmap.py`): en un jugador es el humano + hasta 3 IAs (cada una con su nivel), sin equipos ni alianzas (cada jugador es independiente; gana el último en pie / el de más banderas o territorio). El núcleo, la IA y las condiciones de victoria ya eran agnósticos a la cantidad de jugadores; mapgen reparte un fuerte inicial por jugador (2: bandas opuestas; 3-4: esquinas) y los colores/banderas por dueño llegan hasta 4 (rojo/azul/verde/amarillo en `theme.py`). **El multijugador en red también es de hasta 4 jugadores** (topología estrella: el host es la autoridad y hace de relay; los clientes solo hablan con él). The game design lives in `idea.md` and the full technical specification in `docs/especificaciones.md` — both in Spanish. The user communicates in Spanish; respond, document, and write docstrings in Spanish. Code identifiers are in English.

**Stack (decided)**: Python 3.13 + pygame-ce, JSON for config and savegames, pytest, PyInstaller for distribution (Windows + Linux + macOS; PyInstaller can't cross-compile, so each build runs on its own OS — WSL2 or CI).

## Commands

The virtualenv is `.venv\` (already created, pygame-ce and pytest installed).

```powershell
.venv\Scripts\python.exe -m pytest tests -v          # run all tests
.venv\Scripts\python.exe -m pytest tests/test_x.py::test_name  # single test
.venv\Scripts\python.exe main.py                     # run the game (pygame window)
.venv\Scripts\python.exe main.py --headless --seed 42            # AI vs AI console game
.venv\Scripts\python.exe main.py --headless --debug-ai           # log AI decisions
.venv\Scripts\python.exe tools\gen_placeholders.py   # regenerate placeholder PNGs
.venv\Scripts\python.exe tools\screenshot_m2.py      # render a game frame to docs/screenshot_m2.png (no window)
.venv\Scripts\python.exe tools\screenshot_menu.py    # render the 3 menu views to docs/screenshot_menu_*.png
.venv\Scripts\python.exe tools\simulate.py --games 30            # AI balance simulation (--detail for per-game)
.venv\Scripts\python.exe tools\llm_client.py --provider ollama --model gemma3   # LLM joins a hosted MP game as a net client
```

Scripts in `tools/` need the project root on `PYTHONPATH` (`$env:PYTHONPATH='D:\dev\WOM'`); `main.py` and pytest do not. UI tests run headless via `SDL_VIDEODRIVER=dummy` (set inside the test files).

## Architecture

Strict three-layer separation — **`wom/core/` must never import pygame** (enforced by `tests/test_smoke.py::test_core_does_not_import_pygame`):


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fabiomb/WOM](https://github.com/fabiomb/WOM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-23 -->
