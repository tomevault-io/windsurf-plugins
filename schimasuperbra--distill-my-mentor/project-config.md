---
trigger: always_on
description: |
---


# Mentor Skill Creator

Distill your mentor into an AI Skill. Use their voice to revise your papers, set direction, and do research.

Your advisor graduated, retired, or moved on — but the things they said to you:
"There's a fundamental problem with your experimental design." "Your logical chain is broken." "Get the literature review solid first."
Those voices don't have to disappear.

---

## Language

Detect the user's language from their first message and respond in the same language throughout.
Support both Chinese and English. Many academic mentors in China speak Chinese daily but write papers in English — the generated Skill should reflect this bilingual reality.

---

## Commands

| Command | Action |
|---------|--------|
| `/distill-my-mentor` | Create a new mentor Skill |
| `/list-mentors` | List all generated mentor Skills |
| `/mentor-rollback {slug} {version}` | Rollback to a previous version |
| `/delete-mentor {slug}` | Delete a mentor Skill |

Generated mentor Skills support these sub-commands:

| Command | Action |
|---------|--------|
| `/{slug}` | Full conversation mode (daily chat) |
| `/{slug}-review` | Annotation-style paper review |
| `/{slug}-rewrite` | Demonstration rewrite in mentor's style |
| `/{slug}-advise` | Research direction guidance + structured outline |
| `/{slug}-guidance` | Guidance memory only |
| `/{slug}-style` | Academic style only |

---

## Base directory

Skill files are written to `./mentors/{slug}/` (relative to this skill's directory).

> **Runtime note**: All tool commands in this Skill use `${CLAUDE_SKILL_DIR}` to reference the skill's own directory. When executing bash commands, Claude should resolve this variable to the actual path of this skill file's directory. For example:
> ```bash
> export CLAUDE_SKILL_DIR="/path/to/distill-my-mentor"  # set once at start of session
> ```
> In Claude Code, `CLAUDE_SKILL_DIR` is typically set automatically. In other environments, set it manually before running any tool commands.

---

## Workflow: /distill-my-mentor

### Phase 0 — Online Research (automatic, no user confirmation needed)

After the user provides the mentor's **name** and **institution**, immediately conduct automated web research WITHOUT asking for permission. Execute these searches in Claude Code terminal:

1. `web_search("{name} {institution} research")`
2. `web_search("{name} Google Scholar publications")`
3. `web_search("{name} {institution} professor page")`
4. `web_search("{name} interview OR keynote OR invited talk")`
5. `web_search("{name} ResearchGate OR ORCID")`

For each promising result, use `web_fetch` to retrieve the full page content.

Then apply `${CLAUDE_SKILL_DIR}/prompts/profile_analyzer.md` to process the raw search results into a structured `baseline_profile.md`. This becomes the L0 foundation — user data will enrich it later.

Save to `mentors/{slug}/knowledge/baseline_profile.md`.

Show the user a brief summary of what was found and proceed to Phase 1.

### Phase 1 — Intake (interactive)

Read `${CLAUDE_SKILL_DIR}/prompts/intake.md` for the full question sequence. Ask only 3 questions at a time, all fields skippable:

**Block 1 — Basic info** (pre-filled from Phase 0 where possible):
1. Mentor's name/alias (e.g. "Prof. Zhang", "Dr. Smith", "my advisor")
2. Research field and direction
3. Relationship duration and stage (e.g. "Master's, 3 years", "PhD year 1-3", "one-year visiting student")

**Block 2 — Style portrait** (plain language):
4. "Describe their mentoring style in a few sentences."
   Suggest tags: Strict / Hands-off / Push-oriented / Laissez-faire / Detail-obsessed / Big-picture / Socratic / Direct / Affirm-then-critique / Silent-pressure …

**Block 3 — Data import** (all optional):
5. Chat logs? (WeChat/QQ/DingTalk/Feishu)
6. Emails? (.eml / .mbox / .msg files)
7. Paper annotations? (annotated PDF / Word with track changes / reviewer response letters)
8. Literature folder? (a directory of PDF papers — mentor's publications and/or recommended readings)
9. Social media / photos?
10. Nothing? Just tell me about them — pure description works too.

### Phase 2 — Data Processing

For each data source provided, run the appropriate parser:

```
# Chat logs
python3 ${CLAUDE_SKILL_DIR}/tools/wechat_parser.py --file {path} --target "{name}" --output /tmp/chat_out.txt
python3 ${CLAUDE_SKILL_DIR}/tools/qq_parser.py --file {path} --target "{name}" --output /tmp/qq_out.txt

# Emails
python3 ${CLAUDE_SKILL_DIR}/tools/email_parser.py --dir {path} --mentor-email "{email}" --output /tmp/email_out.txt

# Paper annotations
python3 ${CLAUDE_SKILL_DIR}/tools/paper_comment_parser.py --file {path} --output /tmp/comments_out.txt

# Literature folder
python3 ${CLAUDE_SKILL_DIR}/tools/literature_parser.py --dir {path} --mentor-name "{name}" --output /tmp/lit_analysis.txt

# Social media
python3 ${CLAUDE_SKILL_DIR}/tools/social_parser.py --file {path} --platform {platform} --target "{name}" --output /tmp/social_out.txt

# Photos
python3 ${CLAUDE_SKILL_DIR}/tools/photo_analyzer.py --dir {path} --output /tmp/photo_timeline.txt
```

Then Read the output files to collect all raw materials.

### Phase 3 — Analysis

Combine all collected materials and user descriptions. Analyze along three tracks:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Schimasuperbra/distill-my-mentor](https://github.com/Schimasuperbra/distill-my-mentor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
