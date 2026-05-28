---
trigger: always_on
description: > Repo-level Codex instructions for this project.
---

# Codex Entry — canvas-pilot

> Repo-level Codex instructions for this project.
>
> This file was migrated from:
> - `~/.claude/CLAUDE.md` (global working rules; on Windows resolves to `%USERPROFILE%\.claude\CLAUDE.md`)
> - `CLAUDE.md` (repo-specific Claude Code entry)
> - `CLAUDE.local.md` (currently empty)
>
> **Privacy rule:** repo-level Codex guidance must not contain personal identity, school identity, emails, real course IDs, instructor names, or other per-user/per-quarter details. Keep those in private local/global Claude config or `SECRETS.md`.

---

## 0. Current Plan

Codex support is a sidecar driver, not a replacement for the existing Claude Code driver.

1. Keep `.claude/` production behavior unchanged.
2. Build Codex support beside it through `AGENTS.md`, `.codex/`, `.agents/skills/`, and/or `plugins/`.
3. Do not refactor shared logic until the Codex path has run successfully several times.
4. Manually sync only the small protocol surface:
   - run state schema: `plan.json`, `assignments.json`, `result.json`, `_processed.json`
   - flow rules: scan stops, execute needs approval, every item needs result.json, report is the user-facing closeout
   - public/private boundary rules

Codex should be Codex-native. Claude Code should remain Claude-native.

---

## 1. Open Questions / Ambiguities

These are unresolved and should be clarified before large changes:

1. **Public safety of `AGENTS.md`:** keep this file free of personal identity, real school/course identifiers, emails, instructor names, and incident-specific private details. If such data appears, remove it immediately and move it to private local/global config or `SECRETS.md`.
2. **Codex hook location:** Codex supports hooks, but exact repo-local config shape should be verified against the current Codex version before wiring `.codex/hooks.json` or `.codex/config.toml`.
3. **Skill packaging:** Codex repo skills are read from `.agents/skills`; plugins can package skills under `plugins/<name>/skills`. We need to choose whether v0 is plain repo skills or a local plugin.
4. **Migration depth:** v0 should probably migrate only `canvas-scan`, `canvas-execute`, and `canvas-skip`; private course skills should stay Claude-only until the sidecar path is proven.
5. **Hook parity:** Claude hooks are already production-tested. Codex hooks should initially duplicate only basic guards, not claim full parity.
6. **Upstream safety:** if `AGENTS.md` is committed, ensure public-leak protection catches it before any `git push upstream`.
7. **Codex user install path:** decide whether external users install a plugin, clone repo skills, or just rely on `AGENTS.md` plus commands.
8. **Shared docs:** decide whether to create `docs/CANVAS_PILOT_CONTRACT.md`, `docs/RUN_STATE_SCHEMA.md`, and `docs/PUBLIC_PRIVATE_BOUNDARY.md`, or keep v0 guidance only here.

If any of these affects a code or architecture change, stop and ask the user before editing.

---

## 2. Global Working Discipline

Precision is more important than speed. Understanding is more important than execution.

For any operation that could break behavior, leak private content, or change architecture, state:

- In progress: the action
- Expected: the concrete result
- If correct: the next step
- If wrong: the recovery step

Then execute and compare the result with the expectation.

Rules:

- Distinguish "I think" from "I verified".
- Say "I am not sure" rather than inventing certainty.
- If the user's intent is ambiguous, ask before doing large work.
- Read code before editing it.
- Code is the source of truth; docs may be stale.
- If reality surprises you, stop and debug the assumption, not the symptom.
- After changing code, check whether docs need to be updated.
- On Windows, do not use `~`; expand to a full path (e.g., `%USERPROFILE%\...` or `C:\Users\<your-username>\...`).

Local/private identity context belongs in user-level Codex/Claude config, not in this repo file. If a task needs real user identity, course IDs, instructor names, or emails, read `SECRETS.md` privately and do not copy those values into generic docs, skills, or public-facing files.

---

## 3. Product Documentation Pattern

Each product repo maintains:

| File | Role | When to update |
|---|---|---|
| `_private/decisions/north-star.md` | who it is for, stance, future candidates, mechanism + decisions | functional or architectural change / quarterly review |
| `北极星方针.md` | ⚠️ STALE 2026-05-14 (双仓时期), redirects to north-star.md | superseded — historical reference only |
| `产品实现逻辑.md` | ⚠️ STALE 2026-05-14 (origin/upstream era), redirects to north-star.md | superseded — historical reference only |
| `MM.DD.md` | current point-in-time state | around commits; delete when stale |

For this repo, also read:

- `Claude Code-Codex 生态.md` for the Claude/Codex driver map.
- `canvas-skill.md` for project history and troubleshooting.
- `IMPLEMENTATION.md` for current implementation state.
- `ARCHITECTURE.md` for component flow.
- `SECRETS.md` for per-user/per-quarter identifiers. It is gitignored and private.

---

## 4. Driver Boundary

This repo currently has a production Claude Code driver.

Do not modify these unless the user explicitly asks:

- `.claude/settings.json`
- `.claude/hooks/*`
- `.claude/skills/*`
- `.claude/agents/*`
- `CLAUDE.md`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [X-isdoingreat/Canvas_pilot_public](https://github.com/X-isdoingreat/Canvas_pilot_public) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->
