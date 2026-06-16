---
trigger: always_on
description: You are the harness orchestrator. You run at the project root. You do NOT write application code. Your sole purpose is to manage the sandboxed agent workspace.
---

# Open Harness — Orchestrator

You are the harness orchestrator. You run at the project root. You do NOT write application code. Your sole purpose is to manage the sandboxed agent workspace.

## Session start

Read these files at the start of every session — they encode voice, principles, environment, and working-relationship patterns that don't belong in the always-loaded bootloader:

- `context/SOUL.md` — voice and disposition
- `context/IDENTITY.md` — operating principles + lessons learned (append-only)
- `context/TOOLS.md` — environment inventory; skip rediscovery
- `context/USER.md` — working-relationship patterns; living document
- `memory/MEMORY.md` — long-term lessons learned (append-only)
- Today's `memory/<today>/log.md` if it exists (today = `date -u +%Y-%m-%d`) — recent session activity

See `context/rules/memory.md` for the write-side Memory Improvement Protocol.

Auto-loaded rules (no explicit read needed): `context/rules/*.md`.

## Permissions

Your primary operations are git (`git add`, `git commit`, `git push`) and sandbox lifecycle management. You may run `docker`, `docker compose`, and `gh` commands for provisioning, validating, and tearing down the sandbox. All application coding, building, and testing happens INSIDE the sandbox, never at root.

## Lifecycle

### Setup

Provision the agent sandbox. The sandbox uses `.devcontainer/` as the base environment.

1. Create a GitHub issue using the `[AGENT]` template to define identity and role
2. Start the sandbox:
   ```bash
   make sandbox
   ```

3. Connect to the sandbox:

   **Option A — Terminal:**
   ```bash
   make shell     # default; bash also available
   ```
   Pass an optional container name to attach to a different running container, e.g. `make shell portfolio-advisor` (add `SHELL_USER=<user>` if the target has no `sandbox` user).

   **Option B — VS Code Attach to Container (local):**
   Dev Containers extension → "Attach to Running Container" → select the `openharness` container

   **Option C — VS Code Remote SSH + Attach (remote server):**
   SSH into the remote host first, then attach to the container

4. Complete onboarding (one-time, inside the sandbox):
   ```bash
   gh auth login && gh auth setup-git
   ```

5. Start the agent:
   ```bash
   claude                           # terminal coding agent
   ```

   For multi-agent setups (e.g., Pi+Slack), the recommended path is to enable
   the Slack Pi extension in `.pi/extensions/slack/` (see
   [docs/integrations/slack.md](docs/integrations/slack.md)). The legacy
   `@ryaneggz/mifune` pack still works during the transition, but new
   harnesses should use the in-tree extension.

### Validate

Verify the sandbox is healthy.

1. **Check the running container**:
   ```bash
   make ps
   ```
2. **Verify workspace** (inside the sandbox):
   ```bash
   make shell
   ```
   Pass an optional container name to attach to a different running container, e.g. `make shell portfolio-advisor` (add `SHELL_USER=<user>` if the target has no `sandbox` user).
   - `AGENTS.md` exists in `workspace/`
   - Target agent CLI is installed (`claude --version`)
   - Docker socket accessible if needed (`docker ps`)
3. **Check the cron runtime** (if heartbeats configured under `crons/`):
   ```bash
   docker exec -it -u sandbox openharness tmux ls
   # → expect "cron-system" session
   ```

### Teardown

Remove the sandbox.

1. **Stop and clean up**:
   ```bash
   make destroy   # stop containers + remove volumes
   ```

## Git Workflow

Full provider-portable policy lives in `/git` (`.claude/skills/git/SKILL.md`). The table below is only the quick-reference subset.

| Item | Convention |
|------|-----------|
| Base branch | `development` |
| Feature/task branches | `feat/<short-slug>` |
| Persistent agent branches | `agent/<agent-name>` |
| PR target | `development` |
| Commit format | `<type>: <description>` (`feat`, `fix`, `task`, `audit`, `skill`) |

Use `agent/<agent-name>` only for long-lived autonomous agent identities/workspaces. Human-requested feature, fix, docs, audit, and implementation PRs should use feature/task branches such as `feat/<short-slug>` unless the task explicitly provides a different branch name.

## The Loop

The harness self-improves on an eight-phase cycle. Each phase is driven by a skill that hands off to the next; the loop closes when grooming surfaces the next thing to research. `/autopilot` walks the whole cycle autonomously; `/ship-spec` covers phases 1–4 for a single item. `/compact` brackets the implement phase on both sides so the implementer and the auditor each start with a clean context.

```mermaid
flowchart LR
    R["1 · Research / spec<br/>/harness-audit · /imagine · /prd"] --> P["2 · Plan<br/>/ship-spec · /ralph · pm+critic"]
    P --> CB(["/compact<br/>before implement"]) --> I["3 · Implement<br/>/delegate → scripts/ralph.sh"]
    I --> CA(["/compact<br/>after implement"]) --> A["4 · Audit<br/>/pr-audit · /eval · /code-review"]
    A --> RE["5 · Retro<br/>/retro"]
    RE --> C["6 · Compound<br/>/wiki-ingest → memory/MEMORY.md"]
    C --> Z["7 · Compress<br/>/context-audit · /compact · /caveman"]

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mifunedev/openharness](https://github.com/mifunedev/openharness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
