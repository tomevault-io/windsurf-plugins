---
trigger: always_on
description: Generate Sun audio and stream episodes to Spotify as a podcast. Uses the `sun` CLI (or HTTP API) to authenticate, mint a personal API token, create an audio job from a prompt, fetch episodes incrementally as they finish, and upload each episode sequentially to a freshly-created Spotify show.
---


# sun-to-spotify — Sun Audio Generation + Streaming Spotify Upload

`sun-to-spotify` produces audio through the Sun public API, then streams each finished episode to Spotify as a podcast episode. Given a prompt and a target duration, it creates a job, fetches episodes incrementally as they finish, and uploads each one in order to a Spotify show (a new show is created by default — one Sun audio = one Spotify show).

The skill is built around the `sun` CLI (>=0.2.1 for incremental fetch). For environments where the CLI isn't available, the same flow can run directly against the HTTP API.

> **Naming**: the canonical CLI subcommand is `sun audio`. `sun courses` still works as a hidden alias that prints a one-line deprecation warning on stderr. Use `sun audio` in all new examples.

> **Framing note**: when talking to the user, describe the output as a **podcast, audiobook, or audio course** (whichever fits the topic and duration best — short topical takes lean podcast, long narratives lean audiobook, structured multi-segment lessons lean audio course). Avoid framing this as a "course generator" — it's a versatile audio-experience generator.

## Reference Directory

Load only the file you need — don't inline them.

- [references/cli-usage.md](references/cli-usage.md) — `sun` CLI commands: `login`, `whoami`, `tokens`, `audio`. Install methods, flags (including `--partial` and `--callback-url`), output layout, JSON mode, env-var overrides, troubleshooting.
- [references/http-api.md](references/http-api.md) — HTTP-only flow when the CLI isn't installed: `auth-config` discovery, Supabase password grant, token mint, audio create / status / result (with `?include_partial=true` and `callback_url`), signed-URL audio download, rate-limit headers, error envelope.

---

## Install

The `sun` CLI is independently installable — no monorepo checkout required. Four options, in order of recommendation for external users:

```bash
# 1. curl installer (simplest — picks uv/pipx/pip automatically)
# Download first, then inspect, then run — avoids piping remote scripts directly to a shell.
curl -fsSL https://sunapp-ai.github.io/sun-to-spotify/install.sh -o install.sh
bash install.sh

# 2. uv tool (manual, fastest)
uv tool install 'sun-cli>=0.2.1'

# 3. pipx (isolated)
pipx install 'sun-cli>=0.2.1'

# 4. pip
pip install 'sun-cli>=0.2.1'
```

Verify:

```bash
sun --help
sun --version    # prints "sun-cli <version>"
```

> PyPI package name is `sun-cli`; the installed binary is `sun`. The curl installer is hosted on GitHub Pages from the [`sunapp-ai/sun-to-spotify`](https://github.com/sunapp-ai/sun-to-spotify) repo and requires `uv`, `pipx`, or `pip` to already be available; if none is, it prints install instructions and exits.

If `sun --help` fails after install, ask the user how they installed it before troubleshooting. See [references/cli-usage.md](references/cli-usage.md) for monorepo-dev install (`uv sync` + `uv run sun`) and platform-specific notes.

---

## Core Principles

### Inputs come from the user

Don't invent a prompt. If the user said "make a podcast / audiobook / audio course about X", that's the prompt. If they didn't supply one, ask. Never substitute a creative prompt of your own.

### Save incrementally

Write the `job_id` to disk (or echo it back to the user) immediately after the `202` response. If polling crashes mid-loop, the job keeps generating server-side — re-poll with the same `job_id` rather than restarting.

### Stream — don't wait for the full course

Starting in 0.2.1, episodes finish one-by-one and are visible via `sun audio get --partial`. As soon as episode 1 lands, start uploading to Spotify. Don't block on `SUCCESS` before kicking off Spotify uploads — that wastes minutes per episode and the user loses the streaming benefit.

### Don't cache signed URLs

`audio_url` values are signed for 7 days, but the result endpoint re-signs them on every read. Always fetch fresh URLs from `sun audio get` (which calls `/v1/public/courses/{job_id}`) right before downloading; never persist them.

### Treat `SUCCESS` and `show_uri` as eventually consistent

Two surfaces in this pipeline are eventually consistent. Plan for both — neither is a bug:

- **`sun audio status` returns `SUCCESS` before every episode's `audio_url` is signed.** Generation completes per-episode, then signed URLs land on a slight delay. Keep calling `sun audio get --partial` after `SUCCESS` until every `audio_url` in the manifest is non-null (equivalently: every `NNN-*.mp3` file has appeared under `episodes/`). Don't treat post-`SUCCESS` null `audio_url`s as failures, and don't exit the loop on `SUCCESS` alone — exit only when every episode has been downloaded *and* uploaded.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sunapp-ai/sun-to-spotify](https://github.com/sunapp-ai/sun-to-spotify) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
