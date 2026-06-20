---
trigger: always_on
description: Codex long session, context budget, byte cap, command output protection, needle map, pre-compact raw data, living handoff, retrieval policy, repo workflow governance, PR gate, truncation recovery, large logs, large repo, stale memory, hallucination risk, token budget, oversized /goal, prompt compression, instruction placement. Use to govern evidence retrieval, output limits, handoff, and PR-stage control in long Codex repo/log/data sessions. Do not use for simple one-off tasks, tiny single-file 
---


# Codex Long Session Governance

## Purpose

Use this as a cross-repo Context Budget Governor for long Codex sessions. It limits how Codex retrieves, compresses, reads, outputs, and hands off context.

This Skill does not replace project `AGENTS.md`, repo-specific workflow docs, or business/domain Skills. It governs context handling and repo workflow discipline so those local rules can be read narrowly and followed reliably.

## When to use

Use this Skill when any of the following apply:

- long Codex or `/goal` session;
- staged repo workflow or multi-PR workflow;
- large repo or unfamiliar repo;
- long logs, long Markdown docs, generated reports, checkpoint docs, large CSV, JSON, parquet, or text dumps;
- Codex has shown or may show stale memory, hallucination, context truncation, repeated broad reads, excessive token use, or workflow drift;
- the next action must be derived from repo evidence rather than chat memory;
- the user asks to reduce token burn, manage context, avoid truncation, or create a durable handoff.
- the user asks to shorten an oversized `/goal` or split detailed instructions into repo docs, handoff, controller docs, or task-specific Skills.

## When not to use

Do not use this Skill for:

- one-off small answers;
- tiny edits to one known file;
- tasks where the user explicitly says not to use Skills;
- tasks with no long-context, repo-governance, log-reading, data-reading, PR-gate, or token-budget concern;
- purely creative writing or translation tasks unrelated to repo workflow.

## Desired outcome

The session stays evidence-first, bounded, and resumable. Codex should choose the next action from current repo evidence, cap unknown output, pre-compact raw data and logs, update or propose a living handoff when needed, advance one PR-sized stage at a time, and stop at gates instead of drifting.

For long `/goal` sessions, the launch prompt should stay short and index-like. Detailed requirements should live in the right repo document or Skill, with `/goal` pointing to those sources instead of repeating them.

## Success criteria

- Current repo state is checked before implementation.
- Level 0 and Level 1 context are used before broad reads.
- Unknown or potentially large command output is capped.
- Long files are searched or sampled before any full read.
- Raw data and generated reports are pre-compacted into a needle map before analysis.
- Handoff state is updated or proposed before context becomes stale or oversized.
- Oversized `/goal` prompts are compressed into an index-style controller without dropping hard requirements.
- Detailed stage specs, current state, permanent rules, and historical logs are placed in the right document type.
- Repo work advances one small PR-sized stage at a time.
- PR gates, dirty worktrees, truncation, destructive operations, and high-risk ambiguity stop the workflow.
- Final reports are concise and do not paste full logs, full generated reports, or raw data.

## Inputs and context to collect

Collect only the smallest useful set first:

- repo root, current branch, dirty worktree state, remotes, and recent commits;
- explicit user scope, forbidden paths, and allowed write paths;
- nearest `AGENTS.md` and first-pass handoff or repo-map docs if present;
- current PR gate state if `gh` is available and authorized;
- active stage, target files, expected checks, and stop conditions.

Do not inspect secrets, `.env` content, credentials, SSH keys, API keys, `config.toml`, or unrelated local configuration files.

## Operating principle

Use these invariants:

- Evidence over memory.
- Pre-compact before reading raw data.
- Narrow search before broad read.
- Byte-cap unknown output.
- Summarize; do not paste.
- One PR-sized stage at a time.
- Stop at merge gates.
- Paused External PR Gate State: An open or not-verified-merged PR gate is an
  external wait state. After reporting it once, Codex must pause the active
  goal and wait for explicit user resume. Automatic continuation without a
  user-stated merge/resume/inspect instruction must not query GitHub again,
  must not repeat gate reports, must not mark the goal complete, and must not
  mark the goal blocked merely because the same external PR is still pending.
- If the interface forces a response during the paused external PR gate state,
  return only: `Waiting for PR #X to merge; no checks run.`
- Never merge PRs.
- Do not rely on truncated output.
- Keep `/goal` short; place durable instructions in the right source of truth.
- Update handoff before context becomes stale or oversized.
- Do not create permanent helper files unless the repo scope allows it.

## Prompt Compression And Instruction Placement Policy


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [minqiyang/codex-long-session-governance-skill](https://github.com/minqiyang/codex-long-session-governance-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
