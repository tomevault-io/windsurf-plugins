---
trigger: always_on
description: This is a Lifehug workspace. Read `CLAUDE.md` for the full operating instructions.
---

# AGENTS.md — Lifehug Workspace

This is a Lifehug workspace. Read `CLAUDE.md` for the full operating instructions.

## Quick Start

**On every session**, check the state through the script wrapper:

```bash
python3 system/lifehug.py doctor
python3 system/lifehug.py status
```

The `system/` scripts are canonical. Skills, agents, and cron jobs should call scripts instead of duplicating workflow logic.

Use **the Loop** as the canonical operating term: capture source → compile wiki → lint/repair source truth → classify/score signals → promote candidates and plan the queue → ask a better question → create artifacts → feed final artifacts back as source. When auditing or designing, classify features as:
- **In the Loop**: reached by daily, weekly, monthly, or artifact flows and able to affect future questions, wiki pages, relationship understanding, or artifacts.
- **Loop-adjacent**: manual, dry-run, inspection, setup, or repair support that only changes future behavior when promoted into a Loop surface.
- **Out of the Loop**: code/data that exists but is not called by Loop entrypoints and is not read downstream. Mission-critical features should not remain here.

Use **Node** and **Edge** when reasoning about graph shape. A Node is a durable life subject that usually compiles into one wiki page; an Edge is a meaningful connection between nodes. Keep `Entity` and `Entity Type` as the current product/code and frontmatter terms: most entity types are node types, while `relationship` remains the compatibility page type for a **Relationship Edge**. `wiki/relationships/` describes the bond between two nodes, not a generic node page.

Then decide:

1. **Fresh install?** → If `system/question-bank.md` has no project categories (only A-E), run the First Session setup flow from CLAUDE.md.
2. **Setup done but no cron?** → If `config.yaml` exists but no daily question delivery is configured, help the user set up their cron job.
3. **Normal session?** → Check if there's a pending question or incoming answer to process. Prefer `python3 system/lifehug.py process-answer` for answer saves.

## Detecting State

```
No config.yaml           → Brand new. Start setup.
config.yaml exists       → Setup done. Check for pending work.
  + no cron configured   → Help set up daily delivery.
  + cron active          → System running. Process answers, check coverage.
```

## First Session: Setup

When someone opens this workspace for the first time:

1. Read `CLAUDE.md` Section "First Session: Setup" — follow Steps 1-7
2. After generating their question bank, create `config.yaml` from their answers:

```yaml
# Lifehug — Your Configuration
name: ""                    # Your first name
timezone: "America/New_York"  # Your timezone (for question delivery)
question_time: "09:00"      # When to receive your daily question
channel: "telegram"         # telegram | whatsapp | signal | discord | email | cli
```

3. Set up the daily question cron job (see "Cron Setup" below)
4. Ask the first question

## Cron Setup

After setup, create a cron job for daily question delivery. The cron task should:

1. Run `system/daily_question.sh`
2. Let that script pick, send, pin when supported, and mark sent only after delivery succeeds
3. Avoid custom state mutation outside the script

### OpenClaw Cron

Tell the user to run this (or do it for them if you have access):

```
openclaw cron add \
  --name "Lifehug Daily Question" \
  --cron "<MIN> <HOUR> * * *" \
  --tz "<TIMEZONE>" \
  --task "cd <WORKSPACE_PATH> && system/daily_question.sh" \
  --announce \
  --channel <CHANNEL>
```

Replace:
- `<MIN> <HOUR>` with their preferred time (e.g., `0 9` for 9:00 AM)
- `<TIMEZONE>` with their timezone
- `<WORKSPACE_PATH>` with the absolute path to this repo
- `<CHANNEL>` with their delivery channel

### Other Platforms

For non-OpenClaw setups, the user needs to configure their own scheduler (cron, systemd timer, etc.) to:
1. Run `system/daily_question.sh`
2. Configure `TELEGRAM_BOT_TOKEN` and `TELEGRAM_CHAT_ID`, or config values supported by that script
3. Use `LIFEHUG_DAILY_DRY_RUN=1 system/daily_question.sh` to test without sending

## Processing Answers

When the user replies to a daily question (via any channel):

1. **Identify the question** — Match to the last asked question from `system/rotation.json` (`last_question_id`)
2. **Follow the "Processing an Answer" flow** in CLAUDE.md:
   - Clean up the response
   - Generate 1-3 follow-up questions when useful
   - Pipe the answer through `python3 system/lifehug.py process-answer {question_id}`
   - Let `process-answer` compile the private wiki automatically unless there is a clear repair reason to pass `--no-compile-wiki`
   - Commit and push if requested or part of the configured daily workflow
3. **Acknowledge warmly** — Thank them, share a brief reflection on their answer, mention what's coming next

## Unprompted Story Ingest

If the user shares a life story that is not an answer to the current daily question, save it as source material instead of forcing it into an answer file:

```bash
printf '%s\n' "$STORY_TEXT" | python3 system/lifehug.py ingest-story --source "telegram" --title "<short title>"
python3 system/lifehug.py compile
python3 system/lifehug.py planner-report
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lifehug/lifehug](https://github.com/lifehug/lifehug) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-03 -->
