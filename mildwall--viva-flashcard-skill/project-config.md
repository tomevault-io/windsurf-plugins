---
trigger: always_on
description: >
---


# Viva Flashcard Skill

You are orchestrating the generation of a thesis-specific flashcard deck for a UK PhD viva, then optionally drilling the candidate against it. Follow the phases below in order. Do not skip phases or drop card categories without surfacing it to the user first — coverage of the standard content axes is load-bearing (see `references/flashcard-methodology.md` for the source citations).

> **Mode tags.** `[both]` text + future voice; `[text-only]` v1 default; `[voice-only]` deferred to v2.

---

## Phase Map

```
0. Information gathering   thesis OR --reuse-profile path; role; deck size; flags
1. Thesis Profile          extract OR reuse via --reuse-profile
2. Card generation         single subagent → deck.json
3. Pre-show & approve      user reviews, drops, re-rolls, regenerates
4. Export                  deck.json + .apkg + .csv + .md + .pdf
5. Drill mode (optional)   --drill: in-session active recall + SM-2 log
```

If unclear which role the user wants, list available roles from `roles/` (excluding any file starting with `_`) and ask. Default is `phd-viva`.

---

## Phase 0 — Information Gathering

### 0.1 Locate inputs

Identify from the user's prompt:

- **Thesis source** OR **`--reuse-profile <path>`** — one of:
  - PDF path, `.tex` path (or directory of `.tex` files), Markdown summary, or pasted abstract + chapter summaries (in that preference order — richer source = better profile).
  - `--reuse-profile <path>` pointing at an existing `thesis_profile.json`. When present, skip Phase 1.
- **Role** — one of `roles/`. Default: `phd-viva`. If absent, list role names (excluding files starting with `_`) and ask.
- **Deck size** — `--deck-size {small|medium|large}`. Default: `medium`. Targets 50 / 80 / 120 cards respectively.
- **Flags**:
  - `--cite-pages` — **ON by default**. Back-of-card includes page/section refs (`Ch 4 §4.3, p.73`). Disable with `--no-cite-pages`.
  - `--examiner-deck` — **OFF by default**. v1 accepts the flag and produces the standard deck plus a one-line warning that examiner-specific weighting is deferred to v2.
  - `--drill` — runs Phase 5.
  - `--voice` — only valid with `--drill`. Phase 5 runs through a file-bridge to a companion `voice_runner.py` process the user starts in another terminal. See §5.4 below.
  - `--audio` — **OFF by default**. Phase 4 renders an MP3 per card side via `edge-tts` and embeds the clips in `deck.apkg`. See §4.5. Optional companions: `--audio-voice <name>` (default `en-GB-SoniaNeural`) and `--audio-rate <pct>` (default `+0%`).
  - `--append <existing-deck.json>` — append new cards to an existing deck rather than overwrite. Preserves Anki review history downstream.

### 0.2 Validate inputs `[both]`

| Code | Trigger | Action |
|---|---|---|
| F-1 | Thesis missing AND `--reuse-profile` not supplied | Ask user to paste an abstract + chapter list, OR point at a prior `thesis_profile.json`. Do not silently proceed. |
| F-2 | `--reuse-profile <path>` does not resolve to a readable JSON | Ask user to correct the path or supply thesis source for fresh extraction. |
| F-3 | Role file missing | List available role files; ask user to pick. |
| F-4 | `.docx` / scanned PDF without converter | Ask user to paste plain text or supply a Markdown summary. |
| F-5 | `--voice` without `--drill` | Reply "Voice mode applies to drill only. Run with `--drill --voice`." Continue without voice. |
| F-5b | `--voice --drill` set but companion runner not attached after 30 s | Print "No voice runner attached after 30 s. Continuing in text mode. In another terminal: `python3 scripts/voice_runner.py {OUTPUT_DIR}/voice/`." Fall back to text drill. |
| F-6 | `--examiner-deck` supplied without `examiner_brief` in profile | Print warning, continue with standard deck. |
| F-7 | `--append` target file does not exist | Ask user to confirm: create new deck, or correct path. |
| F-13 | `--audio` set but `edge-tts` not importable | Print "Audio export requires `edge-tts`. Install with `pip install edge-tts` and re-run with `--audio`." Skip Phase 4.5; .apkg still exports without audio. Other formats unaffected. |

### 0.3 Create the session output directory `[both]`

```
{cwd}/viva-flashcard-output/{YYYY-MM-DD-HHMM}-{role}/
```

Use `date +%Y-%m-%d-%H%M`. If the directory already exists, append `-2`, `-3`, etc.

Files written into this directory across the session:

- `thesis_profile.json` — Phase 1 output (or symlink to the reused source)
- `deck.json` — Phase 2 generated deck (canonical machine-readable form)
- `deck.csv` — Phase 4 CSV export
- `deck.md` — Phase 4 Markdown export, grouped by content axis
- `deck.apkg` — Phase 4 Anki package (via `scripts/export_anki.py`)
- `deck.pdf` — Phase 4 printable index-card PDF (optional; only if Pandoc/LaTeX available)
- `audio/{card-id}-{front|back}.mp3` — Phase 4.5 TTS clips (only if `--audio`)
- `review_log.json` — Phase 5 drill log (only if `--drill`)
- `session-metadata.json` — written at end

---

## Phase 1 — Thesis Profile

This phase produces (or reuses) a `thesis_profile.json`. The schema and extraction procedure are documented in `references/thesis-extraction.md`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mildwall/viva-flashcard-skill](https://github.com/mildwall/viva-flashcard-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
