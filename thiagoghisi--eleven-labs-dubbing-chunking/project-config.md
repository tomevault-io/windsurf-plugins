---
trigger: always_on
description: Instructions for Claude Code when working on this repository.
---

# CLAUDE.md

Instructions for Claude Code when working on this repository.

## Project Overview

**dub-chunk** is a Python CLI tool that generates dubbed audio from transcripts using ElevenLabs TTS. It splits transcripts into paragraphs, generates one audio clip per chunk via the ElevenLabs API, and stitches them with natural pauses using ffmpeg.

**Origin:** Extracted from a real project that dubbed a 96-minute Jung-Eissler interview (1953, Library of Congress) from German into English with two cloned voices — 204 API calls, zero consistency issues.

## Architecture

```
src/dub_chunk/
├── cli.py              # Click CLI (generate, parse, estimate, stitch)
├── models.py           # Paragraph, VoiceConfig, TimingEntry dataclasses
├── parsers/            # SRT, labeled text, plain text
│   ├── srt.py
│   ├── labeled_text.py
│   └── plain_text.py
├── consolidate.py      # Merge same-speaker, absorb short interjections
├── clean.py            # Strip /stage directions/, _emphasis_ for TTS
├── timing.py           # Build pause map from word counts
├── tts.py              # ElevenLabs API client + cost estimator
└── stitch.py           # ffmpeg concat with silence gaps
```

Pipeline: `parse → consolidate → clean → timing → TTS (per paragraph) → stitch`

See [docs/architecture.md](docs/architecture.md) for full C4 diagrams.

## Common Commands

```bash
make setup              # Create venv, install deps, verify
make setup-dev          # Editable install for development
make check              # Verify python, ffmpeg, venv, API key
make clean              # Remove venv and build artifacts

# CLI
venv/bin/dub-chunk parse FILE                     # Preview parsed transcript
venv/bin/dub-chunk estimate FILE                  # Cost estimate (no API calls)
venv/bin/dub-chunk generate FILE --voice X=Y      # Full pipeline
venv/bin/dub-chunk generate FILE --voice X=Y --dry-run  # Timing map only
venv/bin/dub-chunk stitch CLIPS_DIR               # Re-stitch existing clips
```

## Development

```bash
make setup-dev          # Editable install (code changes take effect immediately)
make test               # Run pytest
```

After modifying source files, no reinstall needed with `setup-dev`. If you used `make setup` (regular install), run `make deps` again after changes.

## Key Design Decisions

- **One API call per paragraph** — keeps each TTS request short (median 3s, max ~5min) for consistent voice quality. The chunking is what makes long-form dubbing work.
- **ffmpeg for stitching** — subprocess calls, not pydub (avoids audioop dependency removed in Python 3.13+). Silence WAVs generated with raw PCM struct.pack.
- **No translation built in** — user provides already-translated text. Keeps the tool focused and language-agnostic.
- **Resume support** — clips saved as `p{id:04d}.mp3` in clips directory. `--resume` skips existing clips.
- **Clip naming convention** — `p{id:04d}.mp3` (e.g., `p0001.mp3`). Must match between cli.py and stitch.py.

## Testing

```bash
# Parse sample fixtures
venv/bin/dub-chunk parse tests/fixtures/sample_labeled.txt
venv/bin/dub-chunk parse tests/fixtures/sample.srt
venv/bin/dub-chunk parse tests/fixtures/sample_plain.txt

# End-to-end test (requires ELEVENLABS_API_KEY + voice IDs)
venv/bin/dub-chunk generate tests/fixtures/jung_sample.txt \
  --voice "Jung=VOICE_ID" --voice "Eissler=VOICE_ID" \
  --output /tmp/test_dubbed.mp3 -v
```

## Dependencies

- **Python**: click, requests (in requirements.txt)
- **System**: ffmpeg + ffprobe (on PATH)
- **API**: ElevenLabs API key (env var ELEVENLABS_API_KEY)

## See Also

- [Compound Rules](.claude/docs/compound-rules.md) — session learnings and development patterns
- [Outside-In BDD/TDD](.claude/commands/outside-in-bdd-tdd.md) — testing methodology and compound rules OI-001–OI-012

## Gotchas

- `stitch.py` expects clips at `clips_dir / f"p{id:04d}.mp3"` — if clip naming changes in cli.py, update stitch.py too
- `estimate_cost()` in tts.py takes `list[Paragraph]`, not an int — CLI must pass the paragraph list
- `total_duration` CLI param defaults to None — always coerce to `total_duration or 0` before passing to `build_timing_map()`
- Python 3.13+ removed `audioop` module — don't use pydub, use raw ffmpeg subprocess calls
- Generated MP3 clips and final dubbed audio are in .gitignore — they're regenerable

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/thiagoghisi) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
