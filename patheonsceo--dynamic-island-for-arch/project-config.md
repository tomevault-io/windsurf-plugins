---
trigger: always_on
description: **OpenAgentIsland** is an open-source macOS-style **Dynamic Island desktop** for Hyprland, built in **Quickshell/QML** on top of the end-4 (illogical-impulse) framework. The look: **three independent FLOATING islands, no bar** — wallpaper breathes through the gaps. The centerpiece is a morphing notch: minimal clock when idle, expanding for volume, media (with audio visualizer), and — the novel, headline feature — **live Claude Code agent status with permission approval directly from the notch**.
---

# CLAUDE.md — OpenAgentIsland

## What this is
**OpenAgentIsland** is an open-source macOS-style **Dynamic Island desktop** for Hyprland, built in **Quickshell/QML** on top of the end-4 (illogical-impulse) framework. The look: **three independent FLOATING islands, no bar** — wallpaper breathes through the gaps. The centerpiece is a morphing notch: minimal clock when idle, expanding for volume, media (with audio visualizer), and — the novel, headline feature — **live Claude Code agent status with permission approval directly from the notch**.

You are PORTING ideas from a reference project (Hyprfabricated, Fabric/Python+GTK) into our stack (Quickshell/QML). Study the reference for HOW; rebuild in Quickshell. Do not copy Python — translate the technique.

---

## REPO & PATHS (read carefully — this is unusual)

- **THIS REPO / YOUR WORKING DIRECTORY:** `~/Projects/openagentisland/`
  - `quickshell/` — the QML shell code (the actual desktop)
  - `bridge/` — (you create later) the Claude Code agent bridge: hook scripts + socket listener
  - `NOTES.md`, `PROGRESS.md` — design + progress logs (see below)
  - This is a git repo (branch `main`). Commit after each working phase.
- **RUNTIME SYMLINK (don't edit through it — edit the repo):** `~/.config/quickshell/openagentisland` is a SYMLINK → `~/Projects/openagentisland/quickshell`. Quickshell only loads configs from `~/.config/quickshell/<name>/`, so the symlink bridges repo→runtime. `qs -c openagentisland` follows it. **Always edit files in the repo (`~/Projects/openagentisland/quickshell/`).**
- **READ-ONLY REFERENCE (never edit):** `~/Projects/island-reference/hyprfabricated/` — the Fabric project we study.

### HARD RULES (never violate)
1. **NEVER touch `~/.config/quickshell/ii/`** — the user's LIVE desktop. Breaking it breaks their real machine.
2. **Edit ONLY inside `~/Projects/openagentisland/`.** The one exception: Claude Code hook entries in `~/.claude/settings.json` during the agent phase — done carefully and reversibly.
3. **DO NOT merge to `ii`.** The user merges to live config themselves at the end.
4. **User is on `fish`.** `<<EOF` heredocs DON'T work in fish. Write files with your tools, or `printf '%s\n' ...`, or `cat > file` only inside `bash -c '...'`.
5. **Bracketed paste mangles multiline pastes** — prefer writing files directly over asking the user to paste blocks.
6. **Ask before destructive actions; STOP and ask on any UX/design ambiguity.** Don't guess on look/behavior.

---

## NOTES.md — design doc (CREATE & MAINTAIN at repo root)

`~/Projects/openagentisland/NOTES.md` = the **design/architecture reference** ("how it works and why"). Relatively stable — update on architecture decisions, not every edit.

**Create in Phase 0. Must contain:**
- **Reference findings:** after reading `modules/notch.py` + `utils/animator.py`, a concise writeup of Hyprfabricated's notch state model, morph approach, how it composes left/notch/right, and a **Fabric→Quickshell mapping table** (their concept → our equivalent).
- **Architecture:** the three-floating-island design, what each island contains, the full notch state machine (every state, trigger, content), and state precedence.
- **Quickshell/end-4 facts** used (theme tokens, services, panel-family loader, Variants for multi-monitor).
- **Agent bridge design:** socket path, hook events, the JSON schema sent over the socket, the blocking-permission protocol + safety/timeout behavior.
- **Key decisions & rationale.**

NOTES.md is what a new contributor reads to understand the project. Keep it accurate.

## PROGRESS.md — running log (CREATE & MAINTAIN at repo root)

`~/Projects/openagentisland/PROGRESS.md` = the **chronological work log**, updated after every meaningful step.

**Create in Phase 0. Structure:**
- **Current phase & status** at the TOP, always current (which phase, what works, what doesn't).
- **Done:** dated bullets, newest first — what was built + that it was verified in the nested window.
- **Next:** immediate next steps.
- **Blockers / open questions:** anything stuck or needing the user.
- **Gotchas hit:** problems + how solved (so they aren't repeated) — QML errors, API quirks, fish/paste issues.

Update at the end of every session and after each phase. A resuming session (you or the user) should learn exactly where things stand from PROGRESS.md. Be concise but specific, e.g.: "Phase 3 done: volume state morphs cleanly, verified by changing volume in nested window; flicker fixed by watching Audio.volume value, not osdVolumeOpen flag."

---

## DEV WORKFLOW (your "dev server")

A **nested Hyprland-in-a-window** renders `openagentisland` live, isolated from the real desktop — like `npm run dev`.

- Nested config: `~/.config/hypr-nested/hyprland.conf` (minimal Hyprland that launches `qs -c openagentisland`).
- User launches it with:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [patheonsceo/Dynamic-island-for-arch](https://github.com/patheonsceo/Dynamic-island-for-arch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
