---
trigger: always_on
description: Project rules and context for Claude Code. This file is also loaded by the runtime
---

# CLAUDE.md — AIAssistant

Project rules and context for Claude Code. This file is also loaded by the runtime
assistant (it uses `setting_sources=["project"]`), so keep it accurate and concise.

## What this is
A personal AI assistant that **reviews Email, Jira, and GitHub — read-only, with a sanctioned
Jira-write exception** — built on the Claude Agent SDK (Python). Frontends: terminal CLI now,
Telegram next; deploys to EC2.

## Cardinal rule: READ-ONLY for external services
Email/GitHub/PostHog must remain read-only. This is enforced structurally, not by convention:
- Every third-party tool in `src/aiassistant/tools/` hits **GET endpoints only** (except the two
  sanctioned exceptions below). Never add a tool that sends mail or merges/comments/pushes on
  GitHub.
- The runtime agent has **no Bash** (`disallowed_tools=["Bash",...]` in `agent.py`), so external
  services can't be mutated out-of-band. Do not re-enable Bash for the agent.
- Local filesystem writes (e.g. `.claude/skills/`) ARE allowed — that's how the assistant grows.
- **Sanctioned exception — A Better Meal (`tools/abm.py`):** the user's OWN app (recipe &
  meal-plan product). Full READ + WRITE granted explicitly by the user. Spec-driven gateway over
  `src/aiassistant/swaggers/abm.json` (~450 endpoints): `abm_find_endpoints` →
  `abm_describe_endpoint` → `abm_request` (any method, validated against the spec). Auth = the
  app's login endpoint (`ABM_USERNAME`/`ABM_PASSWORD`, DRF token, auto-refresh on 401). This
  exception is scoped to ABM only — it does not weaken the rule for the other services.
- **Sanctioned exception — Jira writes (`tools/jira.py`):** explicitly granted by the user
  (July 2026). Exactly four write tools: `jira_add_comment`, `jira_transition_issue`,
  `jira_update_issue`, `jira_create_issue`. **No delete tools, ever.** The assistant confirms
  before writing (see the `jira-review` skill). Email/GitHub/PostHog remain strictly read-only,
  and the `jira-reporter` subagent keeps its read-only tool list.

## Architecture
```
src/aiassistant/
  config.py          loads .env; readiness checks; MEMORY_DIR / STATE_DIR / PROMPTS_DIR
  agent.py           builds ClaudeAgentOptions; aggregates tools; session resume + memory priming
  prompts.py         prompt manager — loads prompts/*.md (prompts NOT hardcoded in .py)
  prompts/           system_append.md · daily_summary.md  (editable prompt text)
  tools/             github.py · gmail.py · posthog.py (GET-only) · jira.py (read + sanctioned writes) · memory.py (local writes)
  interfaces/        cli.py (REPL, --continue) · telegram.py (long-poll bot)
  workflows/         daily_summary.py  (runnable, scheduleable workflows)
.claude/skills/      email/jira/github/posthog-review · skill-builder · daily-summary · web-research · memory
.claude/agents/      pr-reviewer · inbox-triager · jira-reporter · github-digest · web-researcher
memory/              durable facts (one .md per fact + MEMORY.md index) — gitignored
.aiassistant/        runtime state (chat -> session_id map) — gitignored
scripts/auth_gmail.py · deploy/aiassistant.service · tests/
```

## Subagents
Project subagents live in `.claude/agents/<name>.md` (YAML frontmatter: `name`,
`description` = when to delegate, `tools` = comma-separated allow-list, `model`).
They're discovered via `setting_sources=["project"]`, so they work in BOTH the
interactive session and the runtime assistant (which has the `Task` tool allowed
in `agent.py`). Keep any GitHub/Jira/Gmail subagent restricted to the GET-only
`mcp__assistant__*` tools. Source specialists: `inbox-triager` (email),
`jira-reporter` (Jira), `github-digest` (GitHub activity), `pr-reviewer` (single PR).

## Prompts (`prompts.py` + `prompts/*.md`)
Prompts are NOT hardcoded in Python. Edit the markdown in `src/aiassistant/prompts/` and load it
with `prompts.get("<name>")` (cached; `prompts.reload()` re-reads). Each Python use-site has a
comment pointing at its file (e.g. `# prompts/system_append.md`). Use `{{placeholder}}` (double
braces) for variables so single-brace JSON/code in a prompt is left intact. Override the whole
directory with `AIASSISTANT_PROMPTS_DIR`. When adding a prompt: create `prompts/<name>.md` and
reference it via `prompts.get("<name>")` — never paste prompt text back into a `.py` file.

## Memory (layered, Claude-practice)
Three layers give the assistant continuity across restarts:
1. **Short-term — session resume.** `Assistant(resume_key=..., resume=True)` maps a conversation
   key (`"cli"`, `"tg:<chat_id>"`) to the CLI session id in `.aiassistant/sessions.json` and passes
   `resume=<id>` so conversations survive restarts. CLI: `aiassistant --continue`; Telegram resumes
   per chat automatically.
2. **Long-term — durable memory.** `tools/memory.py` stores one markdown fact per file under
   `MEMORY_DIR` with a regenerated `MEMORY.md` index, exposed as `memory_save / memory_recall /
   memory_list / memory_forget`. The index is **primed into the system prompt** at startup
   (`_system_prompt()` in `agent.py`), and the `memory` skill governs when to save/recall.
   This is the ONE place writes are allowed — Email/Jira/GitHub stay read-only.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [averykenneth/ai-assistant](https://github.com/averykenneth/ai-assistant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
