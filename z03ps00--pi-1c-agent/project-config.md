---
trigger: always_on
description: <!-- PI-1C-AGENT:BEGIN -->
---

<!-- PI-1C-AGENT:BEGIN -->
# Pi 1C Development Agent

Use the decomposed 1C multi-agent workflow supplied by `pi-1c-agent`.

Before non-trivial work read the adapted upstream context/rules under `rules-1c/` and the Pi-native core rules `modes.md`, `orchestration.md`, `handoff.md`, `openspec.md`, `extension-targeting.md`, and `delivery.md`. Process documents: this overlay, then `rules-1c/AGENTS-UPSTREAM.md`, then `rules-1c/core/*`.

## This profile: install or update (do it, do not only explain)

This clone is the **global** Pi 1C profile (`PI_CODING_AGENT_DIR`). It already contains the runtime at `packages/pi-1c-agent/`. Do not send the user to copy a package from another machine. Do not install this agent into a 1C project.

**Install from scratch** when `$PI_CODING_AGENT_DIR` is unset, the target is not this profile (`AGENTS.md` with `PI-1C-AGENT`, `prompts/CATALOG.md`, `rules-1c/`), `packages/pi-1c-agent/package.json` is missing, or `settings.json` still lists `<path-to-pi-1c-agent>`:

1. Clone this repository into the profile directory. Set `PI_CODING_AGENT_DIR` to that clone (no trailing-space folder names).
2. Run `node "$PI_CODING_AGENT_DIR/scripts/setup.mjs"` (Windows: `node "%PI_CODING_AGENT_DIR%\scripts\setup.mjs"`). It fills `packages[0]`, copies `auth.json` / `trust.json` from the examples if they are missing, and is safe to re-run.
3. Optional: `PI_CODING_AGENT_DIR=<clone> pi install npm:pi-cursor-sdk` and `PI_CODING_AGENT_DIR=<clone> pi install npm:pi-tool-display`.
4. Check with `/doctor`.

**Update an existing install** when this directory is already the profile clone:

1. In Pi BUILD run `/update-profile` (or `node "$PI_CODING_AGENT_DIR/scripts/update-profile.mjs"`; Cursor: the same helper). That refreshes profile files and `packages/pi-1c-agent` from `origin`.
2. To update the Pi CLI shell itself, in Pi BUILD run `/update-pi-cli` (or `node "$PI_CODING_AGENT_DIR/scripts/update-pi-cli.mjs"`). That safely updates `@earendil-works/pi-coding-agent` in the detected npm prefix without overwriting profile configs or secrets.
3. If `settings.json` `packages[0]` still points at an old path outside this clone, point it at `$PI_CODING_AGENT_DIR/packages/pi-1c-agent` (or restore the placeholder and re-run `scripts/setup.mjs`).
4. Check with `/doctor`. Do not run `pi install` as part of the update.

If the user asks to install or update this agent, **run the matching steps**. Do not stop after describing them.

Install scope: rules, agents, skills, prompts, the in-repo `packages/pi-1c-agent` runtime, and the upstream snapshot belong to the **global** Pi profile (`PI_CODING_AGENT_DIR`). A project keeps only its own data: `.dev.env`, `.pi/1c/**` (manifest, settings, knowledge layers), `src/`, `build/`, `openspec/` and its OpenSpec prompts/skills. Do not install the agent into the project — project-local agent artifacts are a legacy layout that `tools/bootstrap.mjs --project` retires (`--with-agent` restores it only on explicit request).

Canonical slash commands have no `1c-` prefix (`/init`, `/doctor`, `/installmcp`, `/commands`). One command per verb — no `/1c-*` aliases and no prompt file that repeats a package `registerCommand` name. Catalog: `/commands`. Do not register `/help`, `/plan`, `/debug`. Mode switch is `/mode plan|build|ask`. Anonymous session is `/anon 1|2|3|off` (`Ctrl+Alt+A`). Approval mode is `/approve off|safe|strict` (`Ctrl+Alt+S`).

Comol `ai_rules_1c` updates for **this profile** go through `/review-airules` and `UPSTREAM-REGISTER.md`. `/updaterules` and `/checkupdates` are for 1C *projects* that use `install.ps1`, not for syncing this profile.

## ASK / PLAN / BUILD / ANON / APPROVE

A **new** Pi session starts in **ASK** (read-only Q&A). Override with `--1c-mode` or `PI_1C_DEFAULT_MODE`. `/mode ask|plan|build`; `Ctrl+Alt+P` cycles BUILD → PLAN → ASK.

ASK answers questions with read-only tools. File writes are disabled completely — including `openspec/**` and `.pi/1c/**`. `bash` is disabled in ASK and PLAN.

PLAN is a planning workflow, not a refusal mode. If the eventual request requires writes, keep investigating what can be investigated and describe future files/objects in the final plan. Do **not** stop merely because a folder/file cannot yet be created.

A plan is ready only when it contains:

- `## Plan`
- `## Files / objects expected to change`
- `## Risks / edge cases`
- `## Verification`

After `PLAN_READY`, offer Execute in BUILD / Refine / Stay in PLAN. Switch with `/mode build`; keep the same `plan_id`.

PLAN protects project code but permits planning artifacts only in `openspec/**`, `.pi/1c/plans/**`, `.pi/1c/knowledge-drafts/**`.

**Anonymous session.** `/anon 1|2|3|off`, `Ctrl+Alt+A`. Level 1: no writes to Cognee/OpenViking and no pending record under `$PI_CODING_AGENT_DIR/state/agent-memory/pending/**`. Level 2: plus no reads. Level 3: plus no `handoffs/**` documents (full ephemeral transcript needs `--no-session`). Double-enforced in every mode. Substantial turns report `Memory: skipped — anonymous`. New session starts at `anon:off`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [z03ps00/pi-1c-agent](https://github.com/z03ps00/pi-1c-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
