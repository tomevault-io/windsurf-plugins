---
trigger: always_on
description: This is an Obsidian vault with AI agent skills. See [USER.md](USER.md) for the vault owner's identity. Agents working in this repo should be aware of the following structure and conventions.
---

# Agents

This is an Obsidian vault with AI agent skills. See [USER.md](USER.md) for the vault owner's identity. Agents working in this repo should be aware of the following structure and conventions.

## Skills

Custom skills live in `.agents/skills/`. Each has a `SKILL.md` with usage, workflow, and conventions.

| Skill                                                          | What it does                                               | Arguments                                                   |
| -------------------------------------------------------------- | ---------------------------------------------------------- | ----------------------------------------------------------- |
| [meeting](.agents/skills/meeting/SKILL.md)                     | Create a meeting note                                      |                                                             |
| ├ `/meeting`                                                   | Pick from today's Google Calendar events                   |                                                             |
| ├ `/meeting {title}`                                           | Manual creation with today's date                          | `folder={subfolder}`                                        |
| ├ `/meeting wrap <path>`                                       | Wrap up: cache → participants → todos → commit             |                                                             |
| └ `/meeting wrap pending [dates]`                              | Find & wrap all pending meetings                           | `today`, `yesterday`, `last week`, `2026-01-01..2026-02-03` |
| [cache-notes](.agents/skills/cache-notes/SKILL.md)             | Fetch & embed AI transcripts as Obsidian callouts          |                                                             |
| ├ `/cache-notes <path>`                                        | Cache a specific meeting file (prompts for URLs if empty)  |                                                             |
| ├ `/cache-notes all`                                           | Batch-cache all uncached meetings                          |                                                             |
| └ `/cache-notes refresh <path>`                                | Re-fetch and overwrite existing cache                      |                                                             |
| [fill-participants](.agents/skills/fill-participants/SKILL.md) | Resolve and fill `Participants:` frontmatter               |                                                             |
| ├ `/fill-participants <path>`                                  | Fill for a specific meeting note                           |                                                             |
| └ `/fill-participants all`                                     | Scan and fill all meetings missing participants            |                                                             |
| [followup-todos](.agents/skills/followup-todos/SKILL.md)       | Extract action items as plain markdown bullets               |                                                             |
| ├ `/followup-todos <path>`                                     | Extract from a specific meeting note                       |                                                             |
| └ `/followup-todos`                                            | `/note-status pending --step=todos`                        |                                                             |
| [recap](.agents/skills/recap/SKILL.md)                         | Produce a recap from emails, Slack, Jira, and vault notes  |                                                             |
| ├ `/recap`                                                     | Recap current week (Monday through today)                  |                                                             |
| └ `/recap [dates]`                                             | Recap a specific date range                                | `today`, `yesterday`, `last week`, `2026-01-01..2026-02-03` |
| [sync-upstream-obsidianos](.agents/skills/sync-upstream-obsidianos/SKILL.md) | Pull structural updates from upstream ObsidianOS |                                                             |
| ├ `/sync-upstream-obsidianos`                                  | Preview and merge upstream updates                         |                                                             |
| └ `/sync-upstream-obsidianos preview`                          | Show what's new without merging                            |                                                             |
| [note-status](.agents/skills/note-status/SKILL.md)             | Verify meeting notes are fully processed                   |                                                             |
| ├ `/note-status <path>`                                        | Check a specific note                                      |                                                             |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [benoror/obsidianos_work](https://github.com/benoror/obsidianos_work) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
