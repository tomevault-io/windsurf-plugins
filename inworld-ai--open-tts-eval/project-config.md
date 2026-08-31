---
trigger: always_on
description: This is the operating manual for AI coding agents (Codex, Claude Code, Cursor, …) working in this
---

# Agent guide — sample, evaluate, and report TTS with `tts-assess`

This is the operating manual for AI coding agents (Codex, Claude Code, Cursor, …) working in this
repo. It covers the three workflows the toolkit exists for: **sample** audio from TTS providers,
**evaluate** it against the reference text, and build **reports** (single-run and cross-run).

Golden rules:
- **Never commit API keys or audio.** Keys come from env vars / key files; audio (`*.wav`,
  `*.flac`, `*.mp3`, …) and `tmp/`, virtualenvs, and `.measure_cache/` are git-ignored — keep it
  that way.
- The **evaluation core is offline**; only the *sampling* subsystem calls external provider APIs.
- After code changes, run `ruff check .` and `pytest -q` (both must pass).
- `results.jsonl` is the canonical per-sample record; the HTML reports are derived from it.

---

## Setup

```bash
python3 -m venv .venv && . .venv/bin/activate
pip install -e ".[asr,quality]"     # asr = faster-whisper; quality = torch + NISQA (torchmetrics)
```

Install extras by need: core (`pip install -e .`) is enough for **sampling** and **mock-ASR**
tests; `[asr]` adds real Whisper; `[quality]` adds NISQAv2; `[similarity]` adds ECAPA speaker
similarity; `[all]` = everything.

The CLI entry point is `tts-assess` (module `tts_assess.cli`). Commands: `sample`, `run`,
`compare`, `voices`, `init-config`, `preview`.

---

## Providers & API keys

| provider | key env (default) | default model | notes |
|---|---|---|---|
| `inworld` | `INWORLD_API_KEY` | `inworld-tts-1.5-max` | also `inworld-tts-2`, `inworld-tts-1-max`, `inworld-tts-1` |
| `elevenlabs` | `ELEVENLABS_API_KEY` | `eleven_multilingual_v2` | `eleven_v3`, `eleven_turbo_v2_5` |
| `hume` | `HUME_API_KEY` | `octave-2` | `octave-1`; sits behind Cloudflare (a UA is set for you) |

Provide the key with `--api-key-env NAME` (reads that env var) or `--api-key-file PATH`. List a
provider's prebuilt voices:

```bash
tts-assess voices --provider inworld --api-key-env INWORLD_API_KEY
```

---

## Workflow 1 — Sample

Synthesize a text dataset with chosen voices/models. One run directory per model, named
`<provider>-<model>`, each holding `audio/`, `manifest.jsonl`, `sampling_meta.json`.

```bash
tts-assess sample data/inworld.tts.open_benchmak.en.json \
  --provider inworld --api-key-file ~/inworld.key \
  --model inworld-tts-2 --model inworld-tts-1.5-max \
  --voice Ashley --voice Sarah --voice Oliver \
  --language en-US --format WAV --sample-rate 24000 \
  --limit 20 -o out/samples
```

Key flags: `--provider/-p`, `--model` (repeatable, latest first), `--voice/-v` (repeatable) **or**
`--num-voices N` (+ `--shuffle-voices --seed S` for a seeded diverse pick), `--format`
(WAV/LINEAR16/MP3/FLAC/OGG_OPUS/…; **WAV is the safe default** — `soundfile` must decode it),
`--sample-rate`, `--limit N` (first N texts, for smoke runs), `--concurrency`, `--overwrite`,
`--language`, `--speaking-rate`, `--temperature`.

- Datasets: `.txt` (one utterance/line), `.json`/`.jsonl` (objects or bare strings with a `text`
  field; optional `id`, `language`), or `.csv` with a `text` column. Bundled benchmark:
  `data/inworld.tts.open_benchmak.en.json` (100 messy dialogue utterances).
- Audio is **cached by existence** — re-running resumes; dead keys / quota caps are logged per
  sample and skipped (the run continues). Here "cached by existence" also requires a non-empty
  file and a matching full-request fingerprint; older unfingerprinted rows are refreshed once.
- Voices become `speaker_id` in the manifest.

For a reproducible example of a multi-provider run, use `scripts/sample_multi.py` (pinned voices/models; keys
from env or file; presents `inworld-tts-2` as `inworld-tts-2-preview` via a model alias).
Build your own sampling script to support the new providers, or just provide already sampled audios.


## Workflow 2 — Evaluate

Assess a manifest of `{id, text, audio_path}` (the sampler writes these). Runs ASR → normalize →
WER/CER + audio-health + optional NISQA/speaker/prosody → thresholds.

```bash
tts-assess run out/samples/inworld-inworld-tts-2/manifest.jsonl \
  --config eval.yml -o out/samples/inworld-inworld-tts-2
```

- **Evaluate a run into its own dir** (as above): audio, manifest, results, and report end up
  together, and `audio_path` is stored **relative** → the report's `<audio>` players work when the
  folder is opened or moved. Evaluating into a *different* dir keeps absolute paths.
- **Measurement cache** (`.measure_cache/`, keyed by decoded audio + sample rate/channels + text +
  reference-audio content when used + ASR/normalization/metric config): re-runs are instant
  (`--no-cache` / `--cache-dir` to control). Only thresholds and the report are re-applied each run,
  so tuning bands or the report needs **no** recompute.
- Use `asr.backend: mock` (config) for plumbing tests without downloading Whisper.
- Outputs per run: `results.jsonl`, `summary.json`, `results.csv`, `report_data.json`,
  `report.html`.

## Workflow 3 — Reports & Compare

Every `run` already writes a single-run `report.html`. To compare runs:

```bash
tts-assess compare out/samples/inworld-inworld-tts-2 out/samples/inworld-inworld-tts-1.5-max \

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [inworld-ai/open-tts-eval](https://github.com/inworld-ai/open-tts-eval) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
