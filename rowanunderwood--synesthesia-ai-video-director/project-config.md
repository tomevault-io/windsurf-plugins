---
trigger: always_on
description: Modular Gradio application split across `app.py` (entry point) and supporting modules:
---

# Synesthesia AI Video Director — Project Context

## Architecture

Modular Gradio application split across `app.py` (entry point) and supporting modules:
- **`models.py`** — `ProjectManager` (project I/O, CSV, assets) and `LLMBridge` (LM Studio API)
- **`config.py`** — API endpoints, resolution map, LLM prompt templates, style loading
- **`timeline.py`** — Audio silence analysis, shot generation, LTX frame locking
- **`llm_logic.py`** — Prompt/plot generation orchestration, LLM response parsing
- **`video.py`** — LTX video generation per shot, gallery display, frame count cache
- **`assembly.py`** — moviepy video assembly, cutting room floor compilation
- **`utils.py`** — Frame snapping, base64 image encoding, restart hotkey
- **`ui/`** — Gradio UI split into 6 tabs (project, storyboard, video, assembly, settings, help)

Orchestrates:
- **LM Studio** (local LLM) — generates video prompts and plot summaries via OpenAI-compatible API
- **LTX Desktop** (local AI video engine) — generates video clips from prompts
- **moviepy 1.x** — assembles clips into final video with audio

## Critical: LTX Desktop Resolution Handling

LTX Desktop generates videos at resolutions that are **multiples of 32** for optimal GPU processing. The actual output resolutions do NOT match standard video resolutions, and they **vary depending on whether audio is attached** to the clip (Vocal vs Action shots produce different resolutions at the same preset). For example, 540p without audio = 960x512, but 540p with audio = 960x576.

Because LTX output resolutions are unpredictable, `RESOLUTION_MAP` uses standard resolutions (for UI labels and API requests only). The `assemble_video` function **dynamically detects** the target resolution by reading the first available video clip's actual dimensions. All other clips are resized to match. Do NOT hardcode LTX output resolutions.

## Dependencies

- **moviepy must be < 2.0** — the codebase uses `from moviepy.editor import ...` which was removed in moviepy 2.x. Version is pinned in `requirements.txt`.
- **pydub** requires FFmpeg installed on the system PATH.
- **keyboard** is used for the Ctrl+R restart hotkey.
- **`styles.json`** — optional file in the project root that defines named prompt style presets; loaded at startup by `config.py`.

## Settings Persistence — Rules for New Features

Synesthesia uses a **two-tier settings model**. Every user-adjustable control must fit into one of these tiers; session-only controls should be the rare exception.

### Tier 1 — Global defaults (`global_settings.json`)

Stored by `config.save_global_url_settings()`. Loaded at startup by `config.load_global_url_settings()` into module-level globals. Seeded into every new project's `settings.json` via `config.get_global_defaults()` in `models.create_project()`.

**What belongs here:** API endpoints, wattage/cost settings, any setting defined in `config.GLOBALIZABLE_KEYS` that has been promoted by the user via "📌 Make Current Project Settings Default" (Tab 5).

`config.GLOBALIZABLE_KEYS` is the authoritative whitelist of settings that *can* become global defaults. `config._CODE_DEFAULTS` maps every key in that frozenset to its hardcoded fallback constant.

### Tier 2 — Project settings (`projects/<name>/settings.json`)

Stored by `pm.save_project_settings(dict)` and read by `pm.load_project_settings()`. Each project's file starts as a copy of the current global defaults (seeded on creation) and can diverge freely per project.

**What belongs here:** Everything the user adjusts in Tab 2 (timeline settings, prompt templates, concept/plot), Tab 3 generation preferences (resolution, first-frame mode, style, director, etc.), and any other per-project state.

**What does NOT belong here:** Purely ephemeral UI navigation state (e.g. which shot is selected in a gallery scroll).

### Adding a new UI control — checklist

1. **Decide the tier.** Ask: does this setting make sense as a new-project default? If yes, add its key to `GLOBALIZABLE_KEYS` and its code-level fallback to `_CODE_DEFAULTS` in `config.py`.

2. **Auto-save on change.** Wire a `.change()` (dropdowns/sliders/radios) or `.blur()` (textboxes) handler that calls `pm.save_project_settings({"key": value})`. Tab 2 uses `auto_save_tab2` with a shared inputs list; Tab 3 uses `auto_save_tab3_prefs`. Add new controls to the appropriate function's inputs list and settings dict.

3. **Restore on project load.** Add the value to `handle_load()`'s return tuple in `ui/app.py` using `settings.get("key", fallback)`, and add the corresponding component to the `outputs=[...]` list of `t1["load_btn"].click()`. **The return tuple length must exactly match the outputs list length — Gradio raises a silent error if they differ.**

4. **Restore on project create.** If `handle_create()` resets or populates this control, add it to `handle_create()`'s return tuple and `t1["create_btn"].click(outputs=[...])` in the same position as in `handle_load`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RowanUnderwood/Synesthesia-AI-Video-Director](https://github.com/RowanUnderwood/Synesthesia-AI-Video-Director) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
