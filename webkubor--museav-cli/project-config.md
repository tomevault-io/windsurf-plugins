---
trigger: always_on
description: This CLI is designed to be used directly by coding agents (Claude Code, Codex, etc.), not just humans. Read this before shelling out to `museav`.
---

# AGENTS.md

This CLI is designed to be used directly by coding agents (Claude Code, Codex, etc.), not just humans. Read this before shelling out to `museav`.

## What this is

> 定位（2026-09-16 收口，2026-09-28 修订）：**中台 API 客户端 + 一套本地图像后期工具**。
> 两条红线：① 不内置模型运行时 —— 要本地大模型推理就委托外部工具
> （`reverse --local` → `vlm`/mlx-vlm-kit）；② 不寄生第三方工具 ——
> 跟 MUSE AV 中台无关的能力不进这个 CLI。
> 本地图像那套（抠图/超分/去水印/压缩）是例外且**应当留下**：它不重复任何其它仓库，
> museav-mcp 正是把本 CLI 当能力层在用。
>
> **红线 ② 的例外（2026-09-28 owner 定）**：`skillhub` 回来了。
> 它确实不走在中台 API 上，但它是**出站分发**通道——让本地写好的 Agent Skill
> 一条命令发到小红书 SkillHub，不需要用户另外装 CLI、另外学一套命令。
> 3.6.0 移除它的理由是「一次都没用过」（当时 `whoami` 返回 `loggedIn: false`），
> 那是**没被使用的功能，不是错的定位**。红线 ② 拦的是「寄生」，不是「出站」。
>
> `skillhub` 带来的新义务（比「多一个命令」重）：
> 它会把东西发到**别人的平台**上，而 MUSE AV 里有一批平台公共模板。
> 所以它必须带**出站护栏**，见 `src/skillhub-guard.ts`：
> 平台公共模板的正文不得随 Skill 外发，引用 slug 则放行。改这个命令时别把护栏摘掉。

A command-line client for the "studio" image-generation platform (`https://manager.museav.top`). It generates images from a prompt, reverse-engineers a prompt from an existing image, and lists your own generation history. All output is designed for machine consumption: **stdout carries only the final result** (a URL, a prompt string, or JSON); progress and human-readable info goes to stderr.

## Scope: this tool makes *assets*, not finished videos

Know the boundary before you pick a tool:

| You need | Use |
|---|---|
| An image, or a raw video clip from a model | `museav gen` (add `--video`) |
| Cut out a background / upscale / de-watermark / compress | `museav remove-bg` / `upscale` / `remove-watermark` / `compress` |
| **A finished vertical short video** (images + per-shot captions + BGM/voiceover, templated) | **[reel-kit](https://github.com/webkubor/reel-kit)** — `reel make` |

`museav gen --video` gives you a **silent, caption-less clip** — that's raw material, not
something you post. Assembling material into a publishable short is reel-kit's job: its
layouts are HTML/CSS templates, it does voiceover, and **shot length follows the narration**.

> `museav slideshow` existed in 2.9–2.10 and was **retired in 3.0.0** — it duplicated
> reel-kit. The command still exists as a stub that prints migration instructions.
> Don't try to bring it back; use reel-kit.

## Auth: pick one identity, not both

- **You're acting as an individual user** (a person's own account): `museav login` — opens a device-authorization flow (prints a code + URL, polls until the user approves in a browser). Token cached in `~/.museav.json`, valid 7 days.
- **You're acting as a tenant/service** (no human in the loop, e.g. CI, a backend job): set `STUDIO_API_KEY=sk-studio-xxx` as an environment variable, or run `museav config --apiKey sk-studio-xxx`.

Don't try both — whichever credential is present is what gets used (env var `STUDIO_API_KEY` always wins over the config file). If neither is configured, every command exits 1 with a message telling you which one to set up; that error is your signal to either run `login` interactively (if a human is present to approve it) or ask for an apiKey (if not).

## Core commands

```bash
# Generate an image, wait for it, get the URL on stdout
museav gen --prompt 'a poster, neon lights, cyberpunk' --ratio 9:16

# Generate a video (文生视频/图生视频), wait, get the mp4 URL on stdout.
# There is NO --model flag: which model/upstream to use is the PLATFORM's job — smart
# routing is what the middle platform does. Callers supply prompt + ratio + quality only.
# `museav models` / `museav models --video` are read-only lookups of what the platform is
# currently using, not a selection menu.
museav gen --video --prompt 'a cat stretching on a windowsill, cinematic' --ratio 9:16
museav gen --video --image logo.png --prompt 'logo glows slowly, background fades' --ratio 1:1

# Generate from a pre-configured image template instead of a raw prompt (deterministic
# placeholder substitution server-side, no chat cost). List available templates first —
# the output shows which placeholder keys (if any) each template needs.
museav templates
museav gen --template <id> --fields '{"artist":"name","city":"place"}'

# Create a new image template (tenant-apiKey or platform-admin identity only; a personal
# login gets rejected server-side). Ownership is NOT a flag you pass — the server derives it
# from who's calling: a tenant apiKey auto-attaches its own tenant_id (private to that tenant),
# a platform-admin identity creates a tenant_id=null template shared across all tenants.
# Placeholder keys are auto-extracted from {key} in --prompt if --fields is omitted.
museav templates create --name '演唱会海报' --prompt '{artist} 在 {city} 的演唱会海报' --ratio 9:16

# Reverse-engineer a prompt from an existing image (stdout: English prompt only).
# DEFAULT path is the platform API (the doc used to say "local is primary" — that was
# stale; --local has always been an explicit opt-in flag). With --local the CLI shells
# out to `vlm` (mlx-vlm-kit) and falls back to the API with a warning if it's absent.
# Image URLs always go to the API (the local path takes file paths only).
# This READS the image and nothing else — it will NOT build a template. Passing any

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [webkubor/museav-cli](https://github.com/webkubor/museav-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
