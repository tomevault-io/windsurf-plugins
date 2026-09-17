---
trigger: always_on
description: This project uses OpenWolf for context management. Read and follow .wolf/OPENWOLF.md every session. Check .wolf/cerebrum.md before generating code. Check .wolf/anatomy.md before reading files.
---

# OpenWolf

@.wolf/OPENWOLF.md

This project uses OpenWolf for context management. Read and follow .wolf/OPENWOLF.md every session. Check .wolf/cerebrum.md before generating code. Check .wolf/anatomy.md before reading files.


# XLight AutoSequencer Development Guidelines

Auto-generated from all feature plans. Last updated: 2026-04-22

## Active Technologies
- Python 3.11+ + demucs (new), vamp, librosa, madmom, click, Flask (008-stem-separation)
- JSON files (local filesystem); WAV stem files in `.stems/<md5>/` (008-stem-separation)
- Python 3.11+ + whisperx (faster-whisper + wav2vec2), nltk cmudict, existing deps (vamp, librosa, madmom, demucs, click, Flask) (009-vocal-phoneme-tracks)
- JSON files + `.xtiming` XML files (local filesystem) (009-vocal-phoneme-tracks)
- Python 3.11+ + click 8+, Flask 3+ (existing); no new dependencies (010-analysis-cache-library)
- JSON files — `_analysis.json` (existing, extended with `source_hash`); `~/.xlight/library.json` (new) (010-analysis-cache-library)
- Python 3.11+ + numpy (scoring math), tomllib (TOML config parsing, stdlib in 3.11+), click 8+ (CLI), pytest (testing) (011-quality-score-config)
- TOML files (scoring configs/profiles), JSON files (analysis output with score breakdowns) (011-quality-score-config)
- Python 3.11+ + vamp, numpy, click 8+ (all existing — no new deps) (005-vamp-parameter-tuning)
- JSON files (local filesystem); new `~/.xlight/sweep_configs/` directory (005-vamp-parameter-tuning)
- Python 3.11+ + numpy (signal processing, cross-correlation), librosa 0.10+ (audio features, onset detection), vamp (plugin host), click 8+ (CLI), xml.etree.ElementTree (stdlib, xLights XML export) (012-intelligent-stem-sweep)
- JSON files (analysis output), XML files (`.xtiming`, `.xvc` exports), WAV stem files in `.stems/<md5>/` (012-intelligent-stem-sweep)
- Python 3.11+ + `lyricsgenius` (new optional dep), `mutagen` (new lightweight dep), (013-genius-lyric-segments)
- JSON files — existing MD5-keyed `_analysis.json` cache; `song_structure` field (013-genius-lyric-segments)
- Python 3.11+ + click 8+ (CLI), questionary 2+ (interactive prompts, new), rich 13+ (progress display, new), concurrent.futures (stdlib, parallelism) (014-cli-wizard-pipeline)
- JSON files (existing `_analysis.json` cache, `~/.xlight/library.json`) (014-cli-wizard-pipeline)
- Python 3.11+ + librosa 0.10+, vamp (optional), madmom 0.16+ (optional), demucs/torch (optional), click 8+ (CLI), numpy (016-hierarchy-orchestrator)
- JSON files (hierarchy result), XML files (.xtiming export), WAV stems cached in `.stems/<md5>/` (016-hierarchy-orchestrator)
- Python 3.11+ + `xml.etree.ElementTree` (stdlib), `click` 8+ (existing) (017-xlights-layout-grouping)
- `xlights_rgbeffects.xml` — read and rewritten in-place (backup optional) (017-xlights-layout-grouping)
- Python 3.11+ + `json` (stdlib), `pathlib` (stdlib) — no new dependencies (018-effect-themes-library)
- `src/effects/builtin_effects.json` (built-in catalog), `~/.xlight/custom_effects/*.json` (custom overrides) (018-effect-themes-library)
- Python 3.11+ + `json` (stdlib), `pathlib` (stdlib), `src.effects` (feature 018) (019-effect-themes)
- `src/themes/builtin_themes.json` (built-in), `~/.xlight/custom_themes/*.json` (custom) (019-effect-themes)
- Python 3.11+ + click 8+ (CLI), questionary 2+ (wizard prompts), rich 13+ (progress/tables), mutagen (ID3 tags), xml.etree.ElementTree (stdlib, XSQ generation) (020-sequence-generator)
- `.xsq` XML files (output), JSON analysis cache (existing) (020-sequence-generator)
- Python 3.11+ + librosa 0.10+, vamp, madmom 0.16+, demucs (htdemucs_6s), Flask 3+, click 8+, numpy (021-song-story-tool)
- JSON files (song story output), WAV/MP3 stems in `.stems/<md5>/`, analysis cache (`_hierarchy.json`) (021-song-story-tool)
- Python 3.11+ + pathlib (stdlib), os (stdlib), hashlib (stdlib) — no new dependencies (023-devcontainer-path-resolution)
- JSON files (analysis cache, library index, stem manifests) (023-devcontainer-path-resolution)
- Python 3.11+ (backend), Vanilla JavaScript ES2020+ (frontend) + Flask 3+ (web server), click 8+ (CLI), mutagen (ID3 tags), existing analysis pipeline (027-unified-dashboard)
- JSON files — `~/.xlight/library.json` (song library), `~/.xlight/custom_themes/*.json` (custom themes), `src/themes/builtin_themes.json` (built-in themes, read-only) (027-unified-dashboard)
- Python 3.11+ (backend), Vanilla JavaScript ES2020+ (frontend) + Flask 3+ (web server), click 8+ (CLI), existing analysis pipeline (033-theme-variant-separation)
- JSON files (builtin_themes.json, variant builtins/*.json, custom themes/variants) (033-theme-variant-separation)
- Python 3.11+ (backend), Vanilla JavaScript ES2020+ (frontend) + Flask 3+ (web server), existing `src/generator/plan.py` + `src/generator/xsq_writer.py`, `src/library.py` — no new dependencies (034-library-sequence-gen)
- `~/.xlight/settings.json` (layout path), in-memory `_jobs` dict (generation state), `tempfile.mkdtemp()` (generated `.xsq` files) (034-library-sequence-gen)
- Python 3.11+ + click 8+ (CLI), Flask 3+ (web server), existing generator pipeline (036-focused-effects-repetition)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [derwin12/xlights-autosequencer](https://github.com/derwin12/xlights-autosequencer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-16 -->
