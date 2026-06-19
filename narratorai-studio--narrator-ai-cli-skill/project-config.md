---
trigger: always_on
description: >-
---


# narrator-ai-cli — AI Video Narration CLI Skill

CLI client for [Narrator AI](https://openapi.jieshuo.cn) video narration API. Designed for AI agents and developers.

- **CLI repo**: https://github.com/NarratorAI-Studio/narrator-ai-cli
- **Resources preview** (BGM / dubbing / templates): https://ceex7z9m67.feishu.cn/wiki/WLPnwBysairenFkZDbicZOfKnbc

## Reference Index

This file covers decision flow, the common workflow, and pointers. Detailed lookups live in `references/`:

| Topic | File |
|---|---|
| Resource selection (material / BGM / dubbing / templates) — list commands, response formats, field mapping | `references/resources.md` |
| Full workflow steps with parameter tables and JSON examples (Fast Path + Standard Path) | `references/workflows.md` |
| Magic Video — optional visual template step (catalog, params, language rules) | `references/magic-video.md` |
| Polling pattern, task types, file ops, user account, error codes | `references/operations.md` |

## Pipeline at a Glance

```
                    ┌─── Fast Path (原创文案, cheaper) ───┐
                    │   fast-writing → fast-clip-data     │
  Source material ──┤              ↓                      ├──→ video-composing ──→ (magic-video)
  (material list /  │   [video-composing keys off         │   final MP4 URL       optional visual
   search-movie /   │    fast-clip-data.task_order_num]   │                        template pass
   file upload)     └─────────────────────────────────────┘
                    ┌─── Standard Path (二创文案) ────────┐
                    │   popular-learning → generate-      │
                    │   writing → clip-data               │
                    │              ↓                      │
                    │   [video-composing keys off         │
                    │    generate-writing.task_order_num] │
                    └─────────────────────────────────────┘
```

## Agent Rules (mandatory — apply across all steps)

> **Always:**
> - **Confirm before acting.** Every resource (source, BGM, dubbing, template) and every `magic-video` submission requires explicit user approval. Never auto-select, never auto-submit.
> - **Source data, never invent.** Construct `confirmed_movie_json` from `material list` fields or `task search-movie` output. If neither yields it, ask the user — do not fabricate.
> - **Honor the language chain.** The dubbing voice's language defines the writing task `language` param AND every `magic-video` text param. All three must match. → `references/magic-video.md` § Language Awareness
> - **Paginate `material list` to exhaustion, search programmatically.** Fetch all pages until `total` is consumed, then `grep -i` or `python3 -c` on the JSON. Never trust truncated terminal display.
> - **Poll with the canonical `while` loop at 5-second intervals.** Never use a fixed-iteration `for` loop. → `references/operations.md` § Task Polling
>
> **Never:**
> - **Submit `magic-video` without showing the full request body** (templates + every `template_params` value) and getting user confirmation. The cost is 30 pts/minute and irreversible.
> - **Submit Chinese default values for `magic-video` text params when narration language is non-Chinese.** The defaults are hardcoded Chinese and will appear as Chinese text in a non-Chinese video.
> - **Submit `.task_id` (32-char hex) as `order_num`.** Downstream tasks want `.task_order_num` (the prefixed string like `generate_writing_xxxxx`), not `.task_id`. Submitting the hex returns `10001 任务关联记录数据异常`. The other look-alike — `.results.order_info.order_num` (`script_xxxxx`) — is also wrong; see `references/operations.md` § Task Query Response Shape.
> - **Auto-switch paths after a failure.** If a step fails, surface the error to the user and ask explicitly: retry the same path, switch to the other path, or abort. Never infer a path switch on the agent's own initiative.

## Prerequisites

This skill assumes the `narrator-ai-cli` binary is installed and configured with a valid `NARRATOR_APP_KEY`. See [README.md](README.md) for install / setup. Agents can verify with `narrator-ai-cli user balance`.

## Core Concepts

| Concept | Description |
|---|---|
| **file_id** | 32-char hex string for uploaded files. Via `file upload` or task results |
| **task_id** | 32-char hex string returned on task creation. Poll with `task query` |
| **task_order_num** | Assigned after task creation. Used as `order_num` for downstream tasks |
| **files[]** | Output files in the completed task response (flat, top-level array). Each entry has `file_id`, `file_path`, `suffix`. Read `.files[0].file_id` for the next step's input |
| **learning_model_id** | Narration style model — from a pre-built template (90+) or `popular-learning` result |
| **learning_srt** | Reference SRT file_id. **Mutually exclusive** with `learning_model_id` |

## Conversation Initiation

> ⚠️ **Agent behavior — first message of a session**: Before asking the user for a movie title or workflow path, **proactively orient them** about what the skill offers. Most users assume they need to upload their own video + SRT and don't realize a pre-built material library ships with the skill. Skipping this step often results in unnecessary uploads or aborted sessions.

**Required opening (adapt to the conversation language):**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NarratorAI-Studio/narrator-ai-cli-skill](https://github.com/NarratorAI-Studio/narrator-ai-cli-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
