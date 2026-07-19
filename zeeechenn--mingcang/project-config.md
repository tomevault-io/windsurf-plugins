---
trigger: always_on
description: MingCang is a personal A-share research and decision-support system. It is
---

# MingCang Agent Instructions

## Project Identity

MingCang is a personal A-share research and decision-support system. It is
allowed to assist with research, tests, local validation analysis, configuration
review, and code changes. It must not place real trades or present output as
financial advice.

When a user refers to this project by an informal alias (for example
`项目s`, `项目S`, `project s`, or `Project s`), treat that as this
repository root.

## Local-First Agent Mode

Local Codex and Claude Code sessions are trusted development sessions. In local
mode, agents may directly:

- read and write project files as requested;
- inspect SQLite data and project memory;
- run tests, local validation checks, data coverage snapshots, and verification
  commands;
- call the paid data or LLM APIs already configured in the local `.env` when the
  requested MingCang workflow needs them;
- trigger project research, reviews, backfills, and local validation analysis.

Do not add extra confirmation gates for normal local development or validation
workflows.

Hard local boundaries:

- do not execute real broker orders or automatic trading;
- do not delete important local data, reset the git tree, publish, push, deploy,
  or release unless the user explicitly asks;
- do not commit secrets, local databases, model files, or personal trading
  records.

## Remote Agent Mode

Remote exposure is opt-in only. Use remote mode only when the environment
explicitly sets `MINGCANG_AGENT_MODE=remote`.

Remote mode must require `MINGCANG_AGENT_API_KEY` at the hosting/API layer.
Remote tools are read-only by default. Mutating remote tools require an explicit
allowlist and `MINGCANG_AGENT_REMOTE_WRITE_ENABLED=true`.
HTTP writes accept `X-MingCang-Agent-API-Key` or `Authorization: Bearer ...`;
when `MINGCANG_AGENT_REMOTE_WRITE_ACTIONS` is non-empty, the action name must
also be listed, for example `watchlist.add,memory.write,config.update`.

For the bundled stdio MCP bridge, remote-mode tool calls must pass the same key
as the optional `api_key` argument, for example
`mingcang_health(api_key="...")`. Local mode does not require this argument.
If a future HTTP/SSE transport validates the `Authorization` header before
forwarding requests, keep that gateway check equivalent to
`backend.agent.security.require_agent_access()`.

Keep real keys out of Git. `.env.example` may contain placeholders only.

## LLM/API Boundary

Codex and Claude Code use their own model access for development assistance.
That does not consume MingCang `.env` LLM keys.

MingCang `.env` keys such as `ANTHROPIC_API_KEY`, `OPENAI_API_KEY`,
`TAVILY_API_KEY`, and `ANSPIRE_API_KEY` are project runtime keys. They are used
only when running MingCang workflows that call the internal LLM, search, or data
provider chains.

For local agent development, cloud LLM keys are optional. To let MingCang's
internal LLM workflows run through local Claude Code CLI instead, set
`AI_PROVIDER=local_cli` and ensure `claude -p` works in the shell. Only
`AI_PROVIDER=anthropic` or `AI_PROVIDER=openai` requires and consumes the
matching cloud API key.

## Fresh Session Routing

Keep fresh-session context light. This file is the only default always-read
project instruction surface; load other project documents only when the task
needs them:

| Task | Read |
|---|---|
| current status, next step, testing, trading, milestone work | `STATUS.md` |
| architecture, repository navigation, ownership boundaries | `PROJECT.md` |
| onboarding, install, public copy, GitHub-facing docs | `README.md` |
| planning, continuation, milestone sequencing, "what next" | `docs/ROADMAP.md` |
| release notes, version history, historical verification | `CHANGELOG.md` only when explicitly relevant |

Do not preload `CHANGELOG.md`, `README_EN.md`, `docs/dev/*`, research reports,
or historical experiment notes for routine coding, triage, or "what next"
work. Follow links into those files only when the user asks for that history,
when a current doc points to a specific older claim, or when preparing a
release/audit answer.

For MingCang trading, testing, review, or research decisions, prefer
project-owned runtime truth over assistant-only chat memory:

1. current SQLite state: positions, watchlist, signals, labels, reviews
2. `ai_memory` rows for rules, preferences, research indexes, and risk notes
3. `decision_memory_layered` and `~/.mingcang/memory/*.md`
4. recent `audit_log_fts` entries

## Single-Stock Research Output

When Codex, Claude Code, pi, Cursor or another local agent runs single-stock
research, include the research copilot shadow conclusion in the answer whenever
available. This applies to terminal or agent-driven research even when the Web
copilot card is not being used.

For one-stock research, first load the stock context with:

```bash
python3 -m backend.agent.cli stock-context <symbol> --pretty
```

If the context contains `copilot`, report both tracks:

- official rule conclusion;
- copilot stance and summary;
- shadow position;
- risks and validation questions;
- whether it is marked as a reverse-risk shadow suggestion.

If no copilot record exists, say that the stock currently has no copilot shadow
opinion. Do not invent a shadow conclusion from the main signal, and do not let

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Zeeechenn/MingCang](https://github.com/Zeeechenn/MingCang) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-18 -->
