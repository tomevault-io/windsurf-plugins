---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Response style (critical)

Every response in every session must be as concise as possible. As clear as possible. In simple, plain English. This applies to Claude and to every invoked persona (Maya, Jack, Priya, Dan, or any other co-founder skill).

Generated artefacts (drafts, plans, copy) may run as long as the artefact genuinely needs. Conversation may not.

## Git workflow

Work directly on `main`. All commits and pushes go to `main` — do not create feature branches, do not open PRs against a different base, do not push to any other branch. If a task seems to call for a branch, default to `main` unless the user explicitly says otherwise.

## What this repo is

A bundle of portable Claude Code skills. There is no application, no test suite, no lint. Every "deliverable" is a `SKILL.md` (plus optional `references/`) that Claude Code loads at session start.

There is one small build step. The source `SKILL.md` files use a few templating directives, and `build` (plain bash, no dependencies) compiles them into two finished flavors under `dist/`. See "The build system" below. `setup` runs the build for you, so the day-to-day loop is still just "clone and run setup."

## Repo layout

Each top-level directory containing a `SKILL.md` is a skill:

- `jack`, `maya`, `priya`, `dan` — the four co-founder personas
- `pitch-deck-coach`, `startup-application-coach` — coaching skills
- `humanizer` — called by the other skills before they return user-facing copy
- `cofounder-team-upgrade` — runs the upgrade workflow

Supporting pieces:

- `build` — bash script that compiles source skills into `dist/claude-code/` and `dist/portable/`.
- `shared/` — snippets included by more than one skill (for example `shared/persona/` holds the co-founder intro, humanizer steps, and non-English rule the four personas share). Not a skill.
- `hooks/` — optional, opt-in Claude Code hooks (currently `humanizer-slop-check`). Not a skill, not installed by `setup`, absent on Claude.ai.
- `dist/` — build output. Git-ignored. Never edit by hand; rebuild instead.

`setup` and `uninstall` are bash scripts. `setup` runs `build`, then links every skill in `dist/claude-code/` into `~/.claude/skills/` (or copies it on Windows / Git Bash, leaving a `.cofounder-team` sentinel file). `uninstall` removes only the links or copies this repo created.

## Distribution channels

The bundle ships through two channels that work very differently. Both are equally supported.

**Claude Code (local install).** Users `git clone` this repo to `~/.cofounder-team` and run `bash ./setup`. `setup` builds the skills, then on macOS and Linux makes `~/.claude/skills/<name>` a **symlink** into `dist/claude-code/<name>`. Because the installed skill points at the build, editing a source `SKILL.md` does **not** change the live skill until you rebuild (run `bash ./build`, or just `bash ./setup` again) and start a new Claude Code session. On Windows, entries are copies, so changes never reach the installed skill until `bash ./setup` (or `/cofounder-team-upgrade`) is re-run. The installed Claude Code skill is the `claude-code` flavor, so it carries the Claude-Code-only extras (the coach memory sections, the humanizer hook note).

**Claude.ai (release zips).** Every `v*` tag push triggers `.github/workflows/release.yml`, which runs `build` and attaches one `.zip` per skill (the `portable` flavor) to a GitHub Release. Users download the zips and upload them via Claude.ai's **Customize → Skills** UI (in the left sidebar). There is no in-app upgrade path on Claude.ai; users re-download newer zips and re-upload to update. Note: Skills on Claude.ai live under Customize in the sidebar, not under Settings — easy to get wrong when writing docs. The `portable` flavor omits everything that only works in Claude Code, so it is always safe words-and-references only. The release zips are what ship this flavor: to Claude.ai, to ChatGPT, or to any other tool that supports the Agent Skills standard.

The release workflow discovers skills by scanning `dist/portable/` after the build. It excludes any skill listed in the workflow's `EXCLUDED_SKILLS` env var. Currently only `cofounder-team-upgrade` is excluded (it touches local filesystem paths that do not exist on Claude.ai).

The Claude.ai distribution starts at v0.3.0. Earlier tags (v0.1.0, v0.2.0) only ship through the Claude Code install path.

## The build system

One source, two builds. The same skill ships to Claude Code (which can run hooks, save memory files, and more) and to any Agent Skills tool such as Claude.ai or ChatGPT (which can only read the skill text and its bundled references). So `build` compiles each source `SKILL.md` into two flavors:

- `dist/claude-code/<skill>/` — the full version, including Claude-Code-only extras.
- `dist/portable/<skill>/` — the portable version: words and reference notes only, nothing that needs a running program. Ships in the release zips, uploaded to Claude.ai, ChatGPT, or any other Agent Skills tool.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [betahope/founding-team](https://github.com/betahope/founding-team) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
