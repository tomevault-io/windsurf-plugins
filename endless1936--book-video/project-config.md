---
trigger: always_on
description: This repository is an open-source, natural-language workflow for producing short book videos. Keep reusable code, templates, owned assets, and distilled methods in Git. Keep credentials, private book data, downloaded reference videos, generated episode work, and account data local.
---

# Book Video Agent Guide

This repository is an open-source, natural-language workflow for producing short book videos. Keep reusable code, templates, owned assets, and distilled methods in Git. Keep credentials, private book data, downloaded reference videos, generated episode work, and account data local.

## Startup Checklist

For the first user message in this repository, including a simple greeting such as "你好", run the startup checklist before normal task work unless `.book-automation-state.json` already exists and the user is asking an unrelated repository-maintenance question.

1. Resolve the current repository root with `git rev-parse --show-toplevel`; never hard-code a previous clone path. Check Codex capabilities first: the HyperFrames plugin/Skill and built-in bitmap image generation are capabilities, not user-installed project dependencies. Use them directly when available; do not ask the user to install a separate image model or HyperFrames Skill.
2. Check local runtime prerequisites in one pass by running `node scripts/init.mjs` from the resolved repository root. Do not replace this with an ad-hoc `command -v` plus parsed `--version` output: FFmpeg and FFprobe commonly write version banners to stderr, so a blank stdout is not a missing-command signal. Trust the command exit status and the JSON check result. If Node.js 22+, `npx`, FFmpeg/FFprobe, or `whisper-cli` are missing, report the complete list and ask for one confirmation before installing them. After confirmation, the Agent may install them with the available platform package manager; never install or change system packages silently. The repository does not auto-install these through a project script.
3. HyperFrames runs through `npx hyperframes@0.7.33`, so the first check or render may need npm registry access even when the HyperFrames Skill is available. If `npx` reports `ENETUNREACH`, `EACCES`, `ENOTFOUND`, `ENOTCACHED`, registry access blocked, or a system network policy failure, classify it as an environment/network issue and immediately retry the same check/render with the execution tool's network-capable escalation. Do not ask the user for a second confirmation or ask them to repeat the workflow. Only report the blocker if the escalation itself is denied or fails. Try the local npm cache with `npx --offline` only as a quick fallback; it cannot replace network access when the package is not cached.
4. Check the model at `<repo-root>/assets/models/whisper/ggml-base.bin` using file existence and size, not `ls` output alone; a valid file is at least 100 MB. If missing, ask to download it as part of the confirmed setup, then run `node scripts/download-whisper-model.mjs`. If the download fails, first look for an enabled computer proxy or proxy environment variable; use it for a retry with `--proxy`, asking the user to enable their proxy when none is active. Do not change system network settings silently. If that fails, give the user the browser URL `https://huggingface.co/ggerganov/whisper.cpp/resolve/main/ggml-base.bin`, then install the user-provided file with `node scripts/download-whisper-model.mjs --from "<local-path>"`.
5. Ensure the official Tencent WeChat Reading Skill is installed and enabled through the agent's skill installer when absent; do not ask whether to enable it and do not vendor its source into this repository.
6. If the initialization result reports `wereadApiKey: true` or `weread: enabled`, treat WeChat Reading as configured and never ask for the key again. Only when the result reports it is missing, ask whether to configure the integration. After confirmation, open [微信读书 Skills 官网](https://weread.qq.com/r/weread-skills) with the browser/computer tool and explicitly tell the user: “请在页面获取 API Key，完成后回到本对话把 Key 发给 Agent。” After the user sends it, store it in local `.env` with mode `0600`; never echo or log the key, and never accept it as a command argument. If the user declines key configuration, continue with public research.
7. Run `scripts/init.mjs` after the dependency and Skill checks. It creates local state and the private pipeline file without asking a second WeChat Reading enablement question.

After a body voiceover is supplied, run `node scripts/create-body-timings.mjs "<book>" [script-version]`. When the version is omitted, the Agent resolves it from `brief.json` or the unique version in `script.csv`. It writes Whisper output under the local episode audio folder and creates `body-timings.json` from speech pauses. The default skips the spoken title/author segment; use `--skip-leading 0` when the audio starts directly with the first script line.

Initialization must be idempotent. It must not reinstall a verified skill, overwrite a valid key, reset user choices, or duplicate CSV columns.

## Book Selection

- If the user names a book, use it after verifying title and author.
- If `data/book-pipeline.csv` is missing, header-only, or has no usable candidates, ask one question covering preferred genre, emotional theme, or audience.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Endless1936/book-video](https://github.com/Endless1936/book-video) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-15 -->
