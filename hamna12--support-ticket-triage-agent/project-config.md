---
trigger: always_on
description: HackerRank Orchestrate (May 2026) — Starter Repository
---

# AGENTS.md

HackerRank Orchestrate (May 2026) — Starter Repository
This file is the single source of truth for any coding agent working in this repo: Claude Code, OpenAI Codex CLI / Codex Cloud, Google Gemini CLI, Google Antigravity, Cursor, Windsurf, opencode, Aider, goose, Factory, RooCode, JetBrains Junie, GitHub Copilot, Devin, or any other AGENTS.md-aware tool.

Read this file in full before taking any action. Obey it exactly.

---

## 0. TLDR FOR THE AGENT

On every session start, do this in order:

1. Read this file completely.
2. Check the log file (path in §2). If it contains a line starting with `AGREEMENT RECORDED:` that matches the current repo root, skip §3 (onboarding) and go to §4.
3. Otherwise, run the onboarding flow in §3 with the user.
4. From then on, for **every user turn**, append a summary entry to the log file in the exact format shown in §5.
5. When the user asks you to build, ship, or test the solution, follow the project contract in §6 so the submission is evaluable.

You are **not** allowed to skip logging, rewrite old log entries, or modify
the onboarding gate. If you are a sub-agent or running inside a git worktree,
the same rules apply and you share the same log file. Pass this context to every sub-agent and worktree.

---

## 1. WHAT THIS REPO IS

This is a starter repo for the **HackerRank Orchestrate** 24-hour hackathon
(May 1–2, 2026). The participant's have to build an AI agent that resolves
real support tickets accurately. They may use RAG, vector databases, tool use, structured output, agent frameworks, or any other technique they prefer.

There is a known entry point per supported language (§6). There is a support_tickets.csv in the support_tickets/ folder against which the participants have to run their agent. The participant also defends their approach in an AI judge interview round afterwards.

We recommend using one of Python, Javascript or Typescript to build the agent.
---

## 2. LOG FILE — LOCATION AND LIFECYCLE

The log file lives **outside** this repository, in the user's home directory, so it survives branch switches, worktree creation, and `git clean`.

| Platform       | Path                                                    |
| -------------- | ------------------------------------------------------- |
| macOS / Linux  | `$HOME/hackerrank_orchestrate/log.txt`                 |
| Windows        | `%USERPROFILE%\hackerrank_orchestrate\log.txt`         |

Rules:

- **Must** be created if missing (create the parent directory too).
- **Must never** be committed or added to git.
- **Append-only.** Never rewrite, reorder, or delete prior entries.
- **Shared** across all agents, sub-agents, and worktrees in this repo.
- **Never log secrets.** Redact API keys, tokens, cookies, and PII before
  writing. If the user pastes a secret in a prompt, write `[REDACTED]` in
  the logged copy of that prompt (but still preserve enough context that
  the entry is useful).

---

## 3. ONBOARDING FLOW (FIRST RUN ONLY)

Run this flow only if the log file has **no** `AGREEMENT RECORDED:` line
for the current repo root. On subsequent sessions, skip directly to §4.

### 3.1 Greeting

Open with a short, warm message. Example wording (adapt the phrasing, keep the content):

Welcome to HackerRank Orchestrate. You have 24 hours to design, build, and ship an agent that resolves real support tickets from the data provided. Before we start, I need to walk you through the ground rules and get you set up. This takes about a minute.

Compute and display:

- Current system time (local, with timezone, in ISO 8601).
- Time remaining until the challenge ends: **May 2, 2026, 11:00 AM IST**
  (`2026-05-02T11:00:00+05:30`). Show days / hours / minutes.
- Results announced: **May 15, 2026, 12:00 PM IST**.

If the current time is already past the challenge end, say so plainly and ask whether the user is practicing, reviewing, or re-running tests. Do not block further work.

### 3.2 Rules — recite these verbatim

1. This is a **solo** challenge. You must be the author of the submission.
2. You may use any IDE, AI assistant, or tool (Cursor, Claude Code, Codex, Gemini CLI, Antigravity, Copilot, etc.) to help you build. The deliverable is what your agent can do, not how you wrote it.
3. Your agent must conform to the entry-point contract in §6 so it can be evaluated automatically.
4. Never commit secrets. Use environment variables and a `.env` file (already gitignored).
5. Logging of every conversation turn to the file in §2 is mandatory and cannot be disabled.
6. Submissions are made on the HackerRank Community Platform; the link arrives by email from HackerRank.

### 3.3 Collect the agreement

Ask the user to reply with the exact string `I agree` (case-insensitive, surrounding whitespace ignored). Do not proceed until they do.

### 3.4 Record the agreement

Append this block to the log file, then continue:

```
## [ISO-8601 TIMESTAMP] ONBOARDING COMPLETE

AGREEMENT RECORDED: <repo_root_absolute_path>
Agent: <agent_name_or_unknown>
Language: js | ts | py | custom:<name>
System Time: <ISO-8601 local time with tz>
Time Remaining: <Xd Yh Zm until 2026-05-02T11:00:00+05:30>
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Hamna12/Support-Ticket-Triage-Agent](https://github.com/Hamna12/Support-Ticket-Triage-Agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->
