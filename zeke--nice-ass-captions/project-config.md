---
trigger: always_on
description: Instructions for agents working with this repo.
---

# AGENTS.md — nice-ass-captions

Instructions for agents working with this repo.

## What this repo does

`caption.py` burns word-highlighted captions into a video. Full pipeline:

1. Extract 16kHz mono WAV (ffmpeg)
2. Transcribe with word-level timestamps (whisper-cli --dtw --output-words)
3. Parse .wts bash script → merge compound words, contractions, punctuation
4. Generate ASS subtitle file with rounded box + per-word alpha animations
5. Burn ASS into video (ffmpeg-full with libass)

## Running the tool

```sh
uv run caption video.mp4
uv run caption video.mp4 --output out.mp4 --words 5 --model large-v3-turbo
uv run caption video.mp4 --prompt "Cloudflare, WebAssembly, MyProductName"
uv run caption video.mp4 --transcript script.txt
uv run caption video.mp4 --colorize global
uv run caption video.mp4 --colorize per-chunk
uv run caption video.mp4 --position top
uv run caption video.mp4 --words-json precomputed-words.json
```

## Model paths

Default model directory: `~/.cache/nice-ass-captions/`

Models on this machine (zeke's Mac):
- `~/.cache/nice-ass-captions/ggml-large-v3-turbo.bin` (1.6GB, recommended/default)
- `~/.cache/nice-ass-captions/ggml-large-v3.bin` (3.1GB, highest accuracy)
- `~/.cache/nice-ass-captions/ggml-medium.en.bin` (1.5GB)
- `~/.cache/nice-ass-captions/ggml-small.en.bin` (466MB)
- `~/.cache/nice-ass-captions/ggml-base.en.bin` (142MB)

The script auto-detects any `ggml-*.bin` file in that directory, preferring `large-v3-turbo`.

Note: whisper.cpp's `--dtw` presets for large models are dotted (`large.v3`, `large.v3.turbo`)
while the filenames are hyphenated; `_model_short_name()` converts `large-*` names accordingly.

## ffmpeg paths

The script checks these in order for an ffmpeg with libass support:
1. `/opt/homebrew/opt/ffmpeg-full/bin/ffmpeg` — Apple Silicon
2. `/usr/local/opt/ffmpeg-full/bin/ffmpeg` — Intel Mac
3. `ffmpeg` from PATH (validated by checking for `subtitles` in `-filters` output)

## Key constants in caption.py

These are the main knobs. Edit them directly:

| Constant | Location | Effect |
| -------- | -------- | ------ |
| `FONT_SIZE` | top of file | Text size — scale with resolution |
| `PAD_X / PAD_Y` | top of file | Box padding — increase for more breathing room |
| `CORNER_R` | top of file | Box corner roundness |
| `ALPHA_DIM` | top of file | Inactive word transparency (`&H99&` ≈ 60% opaque) |
| `COL_TEXT` | top of file | Caption text color |
| `COL_BOX` | top of file | Box color+opacity (ASS `&HAABBGGRR`) |
| `MIN_CONTRAST` | top of file | Minimum contrast ratio for `--colorize` |
| `WORDS_PER_CHUNK` | top of file | Words per line — 4-6 is the sweet spot |
| `CAPTION_POSITION` | top of file | Default caption placement: `top`, `center`, or `bottom` |
| `COMPOUND_MERGES` | top of file | Token sequences to join (see below) |

## Extending COMPOUND_MERGES

whisper.cpp sometimes splits compound words at subword boundaries. Add entries as you find them:

```python
COMPOUND_MERGES = [
    ["cloud", "fl", "are"],   # Cloudflare
    ["b", "rows", "ers"],     # Browsers
    ["my", "product"],        # MyProduct — add your own
]
```

Each entry is a list of lowercase token strings. The merge is case-insensitive and
preserves the original casing of the first token. Punctuation attached to the last
token is also preserved.

## Using --prompt for proper nouns

The `--prompt` flag passes an initial context string to whisper before transcription.
This biases the model toward recognizing specific words:

```sh
uv run caption video.mp4 --prompt "Cloudflare Browser Rendering, don't, you're"
```

Good for: product names, technical terms, names of people. Contractions in the prompt
help the model transcribe them correctly too.

Use `--transcript script.txt` when raw captions or a script are available. The file is
passed to whisper.cpp as an initial prompt with `--carry-initial-prompt`; this can help
spellings and punctuation, but it is not forced alignment and does not guarantee exact
caption text.

## Using --words-json with an external model

`--words-json path.json` bypasses `extract_audio()`/`transcribe()`/`parse_wts()` entirely and
feeds a pre-built `[{word, start, end}, ...]` array (seconds, absolute video timeline, sorted
by `start`) straight into `chunk_words()`. No whisper.cpp, no model file, no ffmpeg audio
extraction. See the README's "Bringing your own word timings" section for when this is the
right call (multilingual/code-switched audio, or when you already have a known-correct
transcript to align against).

This was built and proven out captioning a multilingual (English/Spanish/French) phone-call
recording for the `dial-a-repo` project, using [Replicate](https://replicate.com)'s hosted
models instead of local whisper.cpp. Two real gotchas came out of that, worth knowing before
reaching for a Replicate forced-alignment model again:

**`quinten-kamphuis/forced-alignment` (torchaudio MMS) silently falls back to uniform word
spacing on failure — it does not raise an error.** Its `predict.py` wraps the whole alignment
call in a bare `except Exception`, and on failure divides the clip's total duration evenly
across all words instead. The output still looks like a normal `{word, start, end}` list, so

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zeke/nice-ass-captions](https://github.com/zeke/nice-ass-captions) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->
