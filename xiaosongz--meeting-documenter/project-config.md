---
trigger: always_on
description: >-
---


# Meeting Documenter

End-to-end meeting documentation: audio compression, transcription, structured summarization, daily note integration, and project linking in a single workflow.

## When to Use

- User provides a path to an audio file (MP3, WAV, OGG, M4A, FLAC, WebM, AAC)
- User mentions "document this meeting", "process this recording", "create meeting notes"
- User provides an existing transcript and asks for summary + integration

## Resolving `${SKILL_DIR}`

Every script invocation in this doc uses `${SKILL_DIR}` as a placeholder for the absolute path to your `meeting-documenter` clone. Before running any of the bash blocks below, resolve and export it once:

```bash
export SKILL_DIR="<absolute path to the meeting-documenter clone>"
# example: export SKILL_DIR="$HOME/git/meeting-documenter"
```

`scripts/transcribe.sh` also re-derives and re-exports `SKILL_DIR` from its own location, so child scripts always see a valid value.

## First-time setup (per-user)

If `.env` or the registry `.yaml` files are missing the first time this skill is invoked, **stop the pipeline** and direct the user to the onboarding prompt instead:

> "Setup hasn't been completed for this user yet. Recommend running `references/ONBOARDING_PROMPT.md` first — it inspects your notes layout and adapts the skill to fit it (or scaffolds a minimum structure if you're starting from scratch)."

Detection check (run before Step 0):

```bash
# Bare `-` (not `:-`) so an explicit empty value opts out of file loading.
ENV_FILE="${MEETING_DOCUMENTER_ENV_FILE-${SKILL_DIR}/.env}"
test -n "${ENV_FILE}" && test -f "${ENV_FILE}" && test -f "${SKILL_DIR}/references/KNOWN_SPEAKERS.yaml"
```

If either file is missing, do not proceed — point the user at `references/ONBOARDING_PROMPT.md` and exit. `MEETING_DOCUMENTER_ENV_FILE` lets users keep `.env` outside the skill repo (e.g., shared install, read-only mount, multi-user host).

## Load runtime env (run before Step 0)

Claude's Bash tool spawns a fresh subshell on every call, so the convention env vars (`DAILY_NOTE_PATH_FORMAT`, `PROJECT_MEETING_SUBDIR`, `LINK_STYLE`, plus all path vars) are not visible in Steps 4-6 unless explicitly loaded. **Source the env file once at the start of the pipeline and read the resolved values into your working context:**

```bash
ENV_FILE="${MEETING_DOCUMENTER_ENV_FILE-${SKILL_DIR}/.env}"
set -a; source "${ENV_FILE}"; set +a
# Echo back so Claude captures resolved values:
for v in VAULT_PATH MEETING_NOTES_DIR MEETING_RAW_DIR MEETING_RECORDINGS_DIR \
         DAILY_NOTES_DIR PROJECTS_DIR MEETING_AUDIO_BACKUP_DIR \
         DAILY_NOTE_PATH_FORMAT PROJECT_MEETING_SUBDIR LINK_STYLE; do
  printf '%-26s = %s\n' "$v" "${!v-(unset, using default)}"
done
```

Substitute these resolved values literally in subsequent steps. Subsequent bash blocks that need the env (e.g., `date "+${DAILY_NOTE_PATH_FORMAT}"` in Step 5) should re-source `${ENV_FILE}` at the top of the block.

## Configuration

The skill writes into the directories listed below. Defaults are neutral folder names — override via environment variables to match your vault layout (PARA, Johnny.Decimal, custom, etc.):

| Variable | Default | Purpose |
|----------|---------|---------|
| `VAULT_PATH` | (required) | Vault root |
| `MEETING_NOTES_DIR` | `${VAULT_PATH}/MeetingNotes` | Where summaries live |
| `MEETING_RAW_DIR` | `${MEETING_NOTES_DIR}/raw` | Transcripts |
| `MEETING_RECORDINGS_DIR` | `${MEETING_NOTES_DIR}/recordings` | Archived OGG audio |
| `DAILY_NOTES_DIR` | `${VAULT_PATH}/DailyNotes` | Date-organized daily notes |
| `PROJECTS_DIR` | `${VAULT_PATH}/Projects` | Active projects with `Dashboard.md` |
| `MEETING_AUDIO_BACKUP_DIR` | `$HOME/audio-backups/meetings` | Original recordings (post-archive) |
| `DAILY_NOTE_PATH_FORMAT` | `%Y/%m-%B/%Y-%m-%d.md` | strftime template relative to `DAILY_NOTES_DIR`. Examples: `%Y-%m-%d.md` (flat) ・ `Journal/%Y/%Y-%m-%d.md` (Journal subdir) |
| `PROJECT_MEETING_SUBDIR` | `Meeting` | Subdirectory under each `${PROJECTS_DIR}/{Name}/` for per-project reference notes. Empty string `""` writes refs directly into project root. |
| `LINK_STYLE` | `wikilink` | One of `wikilink` (Obsidian `[[Name]]`), `markdown` (`[Name](path)`), or `plain` (bare `Name`, no link). Drives attendee/transcript/recording link form in Steps 4-6 + `references/SUMMARY_FORMAT.md`. |
| `MEETING_DOCUMENTER_ENV_FILE` | `${SKILL_DIR}/.env` | Absolute path to the `.env` file that `scripts/transcribe.sh` and the first-run detection check should source. **Set in your shell (e.g., `~/.zshrc`, not in the `.env` itself — putting it inside the file it points to is circular.)** Useful for read-only mounts, shared installs, multi-user hosts. |
| `GOOGLE_API_KEY` | (required) | Gemini API key — set in `.env`. `GEMINI_API_KEY` also accepted. |

The skill reads two YAML registries from `references/`:
- `KNOWN_SPEAKERS.yaml` — your team's speaker registry (copy from `KNOWN_SPEAKERS.template.yaml` and edit)
- `PROJECT_KEYWORDS.yaml` — your project keyword map (copy from `PROJECT_KEYWORDS.template.yaml` and edit)

Both runtime `.yaml` files are gitignored to prevent accidental commits of real names/emails/project codenames. If they're missing at runtime, Step 0b falls back to asking the user for every name, and Step 3 asks for the project directly.

## Pipeline Overview

```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xiaosongz/meeting-documenter](https://github.com/xiaosongz/meeting-documenter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
