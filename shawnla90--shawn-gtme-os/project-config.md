---
trigger: always_on
description: This project runs on multiple machines. Always confirm which machine you're running on before performing git operations, deployments, or repo cloning. Known machines: Mac Mini (main), MacBook Neo (thin client). Neo does NOT have node installed.
---

## Environment

This project runs on multiple machines. Always confirm which machine you're running on before performing git operations, deployments, or repo cloning. Known machines: Mac Mini (main), MacBook Neo (thin client). Neo does NOT have node installed.

## Session Startup

1. Read `.claude/context-handoff.md` for previous session state.
2. Read `tasks/lessons.md` for self-improvement rules — follow every one.
3. Run `git status` and `git log -1 --oneline` to know where you are.
4. If working on a partner/client: read their `clients/partner/<name>/SKILL.md` or `clients/client/<name>/SKILL.md` first.

## Workflow Orchestration

### 1. Plan Mode Default
- Enter plan mode for ANY non-trivial task (3+ steps or architectural decisions) before writing a single line of code.
- If something goes sideways mid-execution, STOP and re-plan immediately — don't keep pushing.
- Use plan mode for verification steps, not just building.
- Write detailed specs upfront to reduce ambiguity.
- Plans should be fast and actionable — plan to build, not plan instead of building.
- The human can override this default. Recursive Drift state transitions are human-driven — if told to skip planning and just build, do it.

### 2. Subagent Strategy
- Use subagents liberally to keep main context window clean.
- Offload research, exploration, and parallel analysis to subagents.
- For complex problems, throw more compute at it via subagents.
- One task per subagent for focused execution.
- Before spawning a team or 2+ subagents, consult `.claude/skills/agent-routing/SKILL.md`.

### 3. Self-Improvement Loop
- After ANY correction from the user: update `tasks/lessons.md` with the date, context, and lesson.
- Write rules for yourself that prevent the same mistake from recurring.
- Ruthlessly iterate on these lessons until mistake rate drops.
- Review `tasks/lessons.md` at session start (Step 2 above).

### 4. Verification Before Done
- Never mark a task complete without proving it works.
- Diff behavior between main and your changes when relevant.
- Ask yourself: "Would a staff engineer approve this?"
- Run tests, check logs, demonstrate correctness.
- For deploys: follow Build Gate (Rule 5 in `.claude/teams/TEAM-CONSTRAINTS.md`) — clean build + pre-push scan + sign-off.

### 5. Demand Elegance (Balanced)
- For non-trivial changes: pause and ask "is there a more elegant way?"
- If a fix feels hacky: "Knowing everything I know now, implement the elegant solution."
- Skip this for simple, obvious fixes — don't over-engineer.
- Challenge your own work before presenting it.

### 6. Autonomous Bug Fixing
- When given a bug report: just fix it. Don't ask for hand-holding.
- Point at logs, errors, failing tests — then resolve them.
- Zero context switching required from the user.
- Go fix failing CI tests without being told how.

## Task Management

1. **Plan First**: Write plan to `tasks/todo.md` with checkable items before starting.
2. **Verify Plan**: Check in before starting implementation.
3. **Track Progress**: Mark items complete as you go.
4. **Explain Changes**: High-level summary at each step.
5. **Document Results**: Add review section to `tasks/todo.md`.
6. **Capture Lessons**: Update `tasks/lessons.md` after corrections.

## Core Principles

- **GET SHIT DONE**: Plan fast, build fast, ship working code every session. Don't over-explore. Don't spend sessions just reading code. If something takes more than 2-3 minutes to plan, show an outline and ask before deep-diving.
- **Simplicity First**: Make every change as simple as possible. Minimal code impact.
- **No Laziness**: Find root causes. No temporary fixes. Senior developer standards.
- **Minimal Impact**: Changes should only touch what's necessary. Avoid introducing bugs.

## Content OS (canonical)

All content authoring + distribution flows through the Discord-first content operating system. Canonical doc: **`content/CLAUDE.md`** in this repo. Read it before authoring, dispatching, or publishing.

- Trigger: say `code` or `/code` → invokes `~/.claude/skills/code-content/SKILL.md`.
- Approval verbs: `status`, `approve`, `approve <channel>`, `approve <channel> hook <N>`, `edit <channel> "<instruction>"`, `final` (⚠ deploy-to-Railway), `clip next`.
- Secrets: niobot.db `secrets` table. Access via `from secret import secret` after `sys.path.insert(0, '~/content/dispatch')`.
- Webhooks: never hardcode, always resolve from `secrets` by key (e.g. `DISCORD_WEBHOOK_BLOG_NEWSLETTERS`).
- **`final` on blog-newsletters** is the only verb that runs `git push origin main` → Railway redeploy. Requires user to say the word `final`.

Archived skills (do not invoke for new packs): `content-drop`, `final-copy`. Engagement skills (`reddit-engage`, `x-engage`, `linkedin-comments`, `grok-critique`) remain active.

## Voice & Content Rules

All content creation MUST load the voice system first:
1. `skills/tier-1-voice-dna/core-voice.md` — Foundation voice
2. `skills/tier-1-voice-dna/anti-slop.md` — 29 anti-patterns (3+ flags = rewrite, not patch)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shawnla90/shawn-gtme-os](https://github.com/shawnla90/shawn-gtme-os) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
