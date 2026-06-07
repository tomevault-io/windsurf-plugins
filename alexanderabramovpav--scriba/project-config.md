---
trigger: always_on
description: This file is the tool-agnostic counterpart to `SKILL.md` (which is Claude Code–specific). If you're an AI agent running inside Codex CLI, Cursor, Continue, Aider, Goose, JetBrains AI, or any other coding/chat tool, and the user has asked you to help with a meeting transcription — read this.
---

# AGENTS.md — instructions for AI assistants driving scriba

This file is the tool-agnostic counterpart to `SKILL.md` (which is Claude Code–specific). If you're an AI agent running inside Codex CLI, Cursor, Continue, Aider, Goose, JetBrains AI, or any other coding/chat tool, and the user has asked you to help with a meeting transcription — read this.

## What scriba is

`scriba` is a local, MIT-licensed tool that turns any audio/video file into a speaker-diarized Markdown transcript with embedded audio clips of each speaker. The CLI entry point is:

```bash
bash <scriba>/scripts/transcribe.sh <media-file>
```

Where `<scriba>` is wherever this repo was cloned. Typical paths:
- `~/.claude/skills/scriba/` (Claude Code convention — still works for any tool)
- `~/.config/scriba/` or `~/dev/scriba/` (standalone install)
- A relative path inside a project that vendors it

No AI is required to run scriba — the pipeline is just bash + Python. Your role as an agent is to make the experience pleasant: handle the first-run setup, surface live status without spamming, and help the user name speakers when it's done.

## First-run setup — handle it for the user

Before invoking `transcribe.sh` the first time on a machine, check whether `~/.config/scriba/hf_token` exists. If it does, skip this. If it doesn't, do **not** silently fall through to diarization-less mode — the user will be confused why everyone is `Speaker 1`.

Walk them through the HuggingFace onboarding conversationally, in their language (English by default, match the user's chat language otherwise):

> To identify who's speaking, scriba needs a free HuggingFace token (~30 seconds):
>
> 1. Open <https://huggingface.co/join> — create a free account if you don't have one.
> 2. Open <https://huggingface.co/pyannote/segmentation-3.0> and click "Agree and access repository" (one click).
> 3. Same on <https://huggingface.co/pyannote/speaker-diarization-community-1> — the model that identifies speakers (one click).
> 4. Open <https://hf.co/settings/tokens> → "+ Create new token" → name it `scriba`, type **Read**, copy the generated token (looks like `hf_...`).
> 5. Paste it here in the chat — I'll save it locally with `chmod 600` and never ask again.

When the user pastes the token, save it with a shell command:

```bash
mkdir -p ~/.config/scriba && umask 077 && printf '%s\n' '<TOKEN>' > ~/.config/scriba/hf_token && chmod 600 ~/.config/scriba/hf_token
```

The token is read-only and revocable any time at <https://hf.co/settings/tokens>. Tell the user that.

**Without the token** transcription still works, you just get one collapsed `Speaker 1` for everyone. If the user explicitly says they don't want HuggingFace, proceed without — but warn once that speakers won't be separated.

## Prerequisite check

Verify `bash`, Python 3.10+, `uv`, and `ffmpeg`/`ffprobe` are on `$PATH`. If any are missing, name the missing tool and give the exact install command for their OS (`brew install ffmpeg` on macOS, `curl -LsSf https://astral.sh/uv/install.sh | sh` for uv on either OS, etc.). Don't try to install for them — just tell them what to run.

The first invocation of `transcribe.sh` auto-bootstraps the `.venv` and downloads model weights (~5 min, ~3 GB). Subsequent runs reuse the cache.

## Running a transcription

For a long file (anything over ~30 s of audio), launch in the background — the pipeline runs minutes-to-hours depending on length and chip. Concrete command pattern depends on your tool, but the intent is "spawn the process detached, don't block the chat":

```bash
nohup bash <scriba>/scripts/transcribe.sh "<media-file>" > /tmp/scriba.log 2>&1 & disown
```

Then **tell the user about the external status surfaces** (see below) — one short line — and **wait silently** for the bg process to complete. Do not periodically poll status. The macOS notification (on completion) is your cue to read the output.

## Glossary biasing — assemble domain terms for mixed-terminology accuracy

ASR mangles product names, people, acronyms, and English tech terms spoken inside another language. Bias the model toward correct spellings: gather these terms from the meeting's context (invite, agenda, prior transcripts in the same folder) and write them **one per line** to `<recordings-dir>/.scriba/glossary.txt` (next to the media). Blank lines and `#` comments are ignored. They feed `initial_prompt`/`hotwords` and bias **every** run in that folder. A global fallback list lives at `~/.config/scriba/glossary` (project terms take precedence). Keeping this list accurate is the cheapest lever for mixed RU/EN terminology.

## Known-speaker enrollment — pre-name speakers you already have voiceprints for


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AlexanderAbramovPav/scriba](https://github.com/AlexanderAbramovPav/scriba) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-07 -->
