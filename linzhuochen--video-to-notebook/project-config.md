---
trigger: always_on
description: > This file orients AI coding agents (OpenAI Codex CLI, Cursor, Continue, Aider, …) to the video-to-notebook codebase. Claude Code users: see `skills/video-to-notebook/SKILL.md` for the equivalent skill-format walkthrough — the two files cover the same ground.
---

# Agents guide

> This file orients AI coding agents (OpenAI Codex CLI, Cursor, Continue, Aider, …) to the video-to-notebook codebase. Claude Code users: see `skills/video-to-notebook/SKILL.md` for the equivalent skill-format walkthrough — the two files cover the same ground.

## What this repo is

A Python CLI + Astro static-site generator that:

1. **Crawls** YouTube + Bilibili playlists with `yt-dlp` → SQLite.
2. **Tags** transcript chunks with concept labels via Claude (or any agent).
3. **Clusters** proposed tags into a unified ontology.
4. **Synthesizes** a beginner-friendly textbook (one HTML chapter per concept group).
5. **Explains** each concept as a rich illustrated encyclopedia entry.
6. **Builds** the lot into a static Astro site you can host on GitHub Pages.

## Agent-driven workflow (default in v2.3+)

Every LLM stage (`tag`, `cluster`, `curriculum`, `synthesize`, `explain`) runs **in-session by default** — no `ANTHROPIC_API_KEY` required. The CLI writes a JSON prompts envelope to `<state_dir>/prompts/<step>.json` and exits. You read the envelope, reason, write a decisions JSON to the sibling `.decisions.json` path, then re-invoke the same command with `--apply`. The CLI applies your decisions to SQLite.

`tag` and `cluster` also expose an opt-in `--use-api` flag that drives the Anthropic SDK directly (if you have a key). `curriculum`, `synthesize`, `explain` are in-session-only — they have no API path.

The protocol is **agent-agnostic**. Schemas, conventions, idempotency guarantees, error semantics all live in [`docs/AGENT_PROTOCOL.md`](docs/AGENT_PROTOCOL.md). Read that file before driving the pipeline for the first time.

### Quick start for Codex / any agent

```bash
mkdir my-study-site && cd my-study-site
video-to-notebook init
video-to-notebook crawl "<youtube-or-bilibili-url>" --name <slug>
# Repeat crawl for each course

# Then, for each LLM stage:
video-to-notebook <stage> [args]
# CLI writes <state_dir>/prompts/<step>.json and prints a 3-line stderr hint.
# You read the envelope, reason, write <state_dir>/prompts/<step>.decisions.json.
video-to-notebook <stage> [args] --apply
```

For `tag` this is a batch loop (small `--limit`, repeat until `chunks` array empty in the prompts envelope). For `cluster`, `curriculum` it's one-shot. For `synthesize` it's per-chapter; for `explain` it's per-concept.

For a worked end-to-end recipe, see [`examples/frontier-notebook/RUNBOOK.md`](examples/frontier-notebook/RUNBOOK.md).

#### Bilibili requires cookies — playbook for the agent

Bilibili crawls **always** need cookies; an unauthenticated request gets HTTP 412 before yt-dlp can even list a playlist. The full playbook is in [`skills/video-to-notebook/SKILL.md`](skills/video-to-notebook/SKILL.md#bilibili-cookies-playbook-required--bilibili-always-needs-cookies). Short version:

1. Try `--cookies-from chrome` (or firefox / safari / edge) first. On Linux this usually works. On macOS, Chrome v10 cookies are Keychain-encrypted and yt-dlp typically can't read them.
2. If you see `HTTP 412`, `cannot decrypt v10 cookies`, or `expected string or bytes-like object, got 'bool'`, **stop retrying** — those failures are deterministic. Switch to step 3.
3. Ask the user to install the **"Get cookies.txt LOCALLY"** browser extension, log into bilibili.com, export Netscape-format cookies, and save the file **outside** `~/Downloads/`, `~/Desktop/`, and the root of `~/Documents/` (those are TCC-protected on macOS and yt-dlp can't read them). Then run with `--cookies-file <absolute-path>`.

For videos without official subtitles, also add `--whisper` to fall back to local transcription.

### Agent identifier

When you write a results envelope, set the agent-id field (`tagger_model_id`, `reviewer_model_id`, `designer`, `synthesizer`, `explainer`) so the DB records which agent produced each decision. Convention:

| Agent             | Identifier              |
|-------------------|-------------------------|
| Anthropic API     | `claude-haiku-4-5`, etc. (literal model id) |
| Claude Code       | `claude-code-max:v1`    |
| OpenAI Codex CLI  | `codex-cli:v1`          |
| Cursor / Continue | `cursor:v1` / `continue:v1` |

Free-form strings. Pick something distinct so future audits can attribute decisions.

## Codebase layout

```
src/video_to_notebook/
├── cli.py              # typer entrypoint
├── config.py           # PROJECT_MARKER, find_project_root
├── crawl/              # yt-dlp adapters: YouTube, Bilibili
├── tag/                # Claude Haiku per-chunk tagging
├── cluster/            # MiniLM embeddings + LLM-reviewed merges
├── curriculum/         # chapter sequence design
├── synthesize/         # per-chapter HTML generator
├── explain/            # per-concept HTML explainer (v1.3+)
├── build/              # SQLite → Astro content collections
└── db/                 # session.py + migrations/*.sql

template-site/          # Astro 5 site template (copied to project on `init`)
skills/video-to-notebook/   # Claude Code skill (parallel to this AGENTS.md)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LinZhuoChen/video-to-notebook](https://github.com/LinZhuoChen/video-to-notebook) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
