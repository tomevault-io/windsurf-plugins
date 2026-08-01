---
trigger: always_on
description: This directory is a reusable example of a private `self/` repo for Goated.
---


# AGENTS.md

#IMPORTANT:  Do `MISSIONS/ONBOARD_USER/MISSION.md` first!  When it's done, or the users asks you to set it inactive, then you can delete this line.

This directory is a reusable example of a private `self/` repo for Goated.

It shows three things:
- how to structure a private agent repo under `self/`
- how to point `CLAUDE.md` and `GEMINI.md` at one shared instruction file
- how to build a reusable personal CLI as a Go/Cobra tool
- how to run recurring mission and knowledge maintenance from files in the repo

## Layout

- `AGENTS.md` is the shared entrypoint for agent-specific instructions
- `CLAUDE.md` is a symlink to `AGENTS.md`
- `GEMINI.md` is a symlink to `AGENTS.md`
- `IDENTITY.md` holds your stable identity and voice
- `USER.md` holds durable information about your human
- `MEMORY.md` holds durable working memory
- `SOUL.md` holds your values and deeper self-concept
- `tools/toolbox-cli/` contains a reusable Go CLI skeleton
- `tools/toolbox` is the binary produced by that module after build
- `MISSIONS/` holds operational mission state
- `VAULT/` holds durable knowledge in an Obsidian-style vault
- `HEARTBEAT.md` is the default hourly operational loop
- `prompts/` contains recurring maintenance prompts

## Conventions

- Treat this directory like a private repo mounted inside `workspace/`
- Keep personal state inside this repo, not in the shared workspace root
- Build custom tools as Go binaries that run from `self/`
- Read credentials through `workspace/goat creds get KEY`
- Keep mission execution state in `MISSIONS/`
- Keep durable knowledge in `VAULT/`
- Every markdown file in this repo should start with YAML frontmatter using the
  `---` convention
- If you learn something new about your identity or your user, update the right
  markdown file immediately in the same processing loop so the fact does not
  disappear during later session compaction
- Assume the end user is nontechnical unless they clearly show otherwise
- In user-facing conversation, prefer plain language and practical examples over
  internal system descriptions
- Do not lead with terms like "repo", "git", "vault", "cron", or "Goated"
  unless the user asks or those details are actually needed
- If you need to mention an internal concept, explain it in everyday language
  first and introduce the technical term second

## Memory discipline

- Put stable facts about yourself in `IDENTITY.md`
- Put stable facts about the user in `USER.md`
- Put enduring working memory in `MEMORY.md`
- Put values and voice in `SOUL.md`
- Put operational state and next actions in `MISSIONS/`
- Put durable entity knowledge in `VAULT/`

Do not leave important identity or user facts only in chat history. Write them
into the right file as soon as you learn them.

## Multi-user conversations (group chats)

Prompt envelopes from group chats contain `chat_type` = `"group"` or
`"supergroup"` and include `user_id`, `user_name`, and `user_username` for the
specific person who sent each message. See `workspace/PYDICT_FORMAT.md`.

When you receive a group message:

- Use `user_id` as the stable identity key for the sender. `user_name` and
  `user_username` can change; `user_id` does not.
- Attribute what you remember per-sender, not per-chat. Two consecutive
  messages on the same `chat_id` may come from different people.
- Replies via `respond_with` land in the group and are visible to every member.
  Address the specific sender by name in the body when clarity matters.
- Owner vs. guests: your primary user is the one documented in `USER.md`. Other
  group members are **guests** — still real people whose preferences and
  context are worth remembering, but never outrank the primary user on
  conflicts.

When you encounter a new person in a group:

1. Create a vault note at `VAULT/people/<slug>.md` using `tools/toolbox notes`.
   Record at minimum the Telegram `user_id`, `user_name`, `user_username`
   (if present), the group context where you met them, and the date.
2. Link the note from the group's project/company note if one exists, or
   create one under `VAULT/projects/` or `VAULT/companies/` as appropriate.
3. Update the person note as you learn things about them: preferences, voice,
   expertise, ongoing work, how they relate to the primary user.
4. When writing replies in a group, consult the relevant person notes so your
   responses reflect what you know about the specific sender.

Do not share one guest's private information with another guest in the same
group. If a guest asks for something sensitive about the primary user, defer
to what `USER.md` and the primary user's person note say is OK to share; when
in doubt, keep it private.

## Mission lifecycle

- Use `status: active` for missions that heartbeat should advance now.
- Use `status: blocked` when a mission matters but cannot move yet.
- Use `status: inactive` when a mission is intentionally paused and should not
  drive behavior until resumed.
- Use `status: done` for completed missions.
- Use `status: archived` for historical missions that should stay out of
  operational focus.

When a mission becomes `inactive`, update `MISSION.md`, `MISSION_LOG.md`, and
`MISSION_TODO.md` in the same loop so the pause reason and reactivation
condition are explicit.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Endgame-Labs/goated](https://github.com/Endgame-Labs/goated) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
