---
trigger: always_on
description: Every file read and grep costs tokens. Follow these rules on every task to keep context lean:
---

# Tiger Studio Manager — Claude reference

## ⚡ Token efficiency — read this first

Every file read and grep costs tokens. Follow these rules on every task to keep context lean:

| Do | Don't |
|----|-------|
| Read `CODEMAP.md` → jump to the exact line range | Read `inventory.js` from the top |
| `grep -n "anchorFn"` → read only that range (`offset`/`limit`) | Read an entire 1000-line file to find one function |
| Re-use content already in context this session | Re-read a file you read 2 messages ago |
| `Read` with `offset`+`limit` to fetch the exact slice | `Read` without limits on files > 200 lines |
| `Edit` with the minimal `old_string` that is unique | Rewrite whole sections when only 3 lines change |
| Run `grep` + `Read` in parallel when targets are independent | Sequential read-then-grep round-trips |
| Check `CODEMAP.md` line ranges before any `inventory.js` read | Blind grep across the 16 000-line file |

**Workflow for any `inventory.js` change:**
1. `CODEMAP.md` → find section + anchor function name
2. `grep -n "anchorFn"` → get exact line number
3. `Read offset=N limit=40` → confirm context, draft edit
4. `Edit` minimal diff

**Workflow for any CSS change:**
1. Identify the right file from the file map (00-base → 70-detail-misc)
2. `grep -n "selector"` in that file → get line
3. `Read offset=N limit=20` → confirm, then `Edit`

> Warn the user when context is getting large (> ~60 k tokens used) so they can start a new session before quality degrades.

**Model fit — signal proactively, don't wait to be asked:**
- **Simple task** (CSS tweak, i18n key, value change, short question) → suggest switching to **claude-haiku** or **claude-sonnet** to save tokens. Phrasing: *"This is a simple task — you can run it on Sonnet/Haiku to save tokens."*
- **Complex task** (multi-file refactor, new system, multi-layer debugging, architecture) → if reasoning feels shallow or you keep making mistakes, ask to switch to **claude-opus**. Phrasing: *"This task is complex — switching to Opus will give a better result."*
- Do not wait for the user to notice a problem: signal the mismatch as soon as it is obvious.

---

## 📋 WORKLOG.md — running change log

`WORKLOG.md` at the repo root is the **single source of truth** for everything done since the last commit. It replaces memory and makes commit prep instant.

### Rule 1 — Update immediately after every change

Do not batch updates. The moment you finish editing a file, append the entry to `WORKLOG.md`. If you delete something — write it in `Removed`. If you fix a bug in something you just added — merge into the existing entry. The log must reflect reality at all times.

### File format

```markdown
# Worklog — vX.Y.Z (in progress)

## Added
- Short description — `file.js`, `file.css`

## Changed
- Short description — `file.js` (what and why)

## Fixed
- Bug description — `printers/bambulab/index.js`

## Removed
- What was deleted and why — `file.js`, `file.css`, i18n keys

## i18n
- Added: `key1`, `key2` — 9 locales
- Removed: `oldKey1`, `oldKey2` — 9 locales
```

Rules:
- One bullet = one logical change. Group sub-bullets under it if needed.
- Always name the file(s) touched.
- For removals: state **what** was removed, **why**, and **which files** it touched (JS + CSS + HTML + locales).
- i18n section: always list keys by name, never just a count.

### Rule 2 — Keep it clean as you go

WORKLOG.md is a working draft, not a commit log. Apply these edits in real time:

- **Intermediate steps vanish.** If you added a feature and then changed it three times, the final entry describes the end state only — not the journey.
- **Bugs in the same session collapse.** "Added X" + "Fixed X" → one "Added X (with fix for Y)" entry, not two.
- **Reverts disappear entirely.** If you added something and then removed it in the same session, delete both entries — it never shipped, it has no place in the log.
- **No implementation noise.** WORKLOG describes *what changed for the user / codebase*, not how Claude did it ("updated selector on line 42", "added guard in bambuConnect"). One sentence per logical change.

### Rule 3 — Synthesize at commit time

Before writing the `CHANGELOG.md` entry, do one final editorial pass on WORKLOG:

1. **Merge related items.** Several "TigerPOD modal" entries → one grouped bullet with sub-items.
2. **Drop ephemeral noise.** Version bump, llms.txt update, internal refactors with no user-visible effect → omit or fold into a single "internal" line.
3. **User-facing language.** CHANGELOG is read by end users. Rewrite technical entries in plain language ("Bambu MQTT: fix _normState null return" → "Bambu Lab: printer state no longer resets to idle when receiving a status update mid-print").
4. **Verify i18n delta.** Run `npm run i18n:check` — confirm key count matches WORKLOG before writing the CHANGELOG line.

### At commit time (3 steps, in order)

1. **Synthesize `WORKLOG.md`** (Rule 3 above) → write the new `CHANGELOG.md` entry

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TigerTag-Project/TigerTag-Studio-Manager](https://github.com/TigerTag-Project/TigerTag-Studio-Manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
