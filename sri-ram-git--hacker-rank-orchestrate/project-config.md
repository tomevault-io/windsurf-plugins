---
trigger: always_on
description: HackerRank Orchestrate (June 2026) — Multi-Modal Evidence Review
---

# AGENTS.md

HackerRank Orchestrate (June 2026) — Multi-Modal Evidence Review

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

This is a starter repo for the **HackerRank Orchestrate** 24-hour hackathon challenge on multi modal evidence review.

Participants must build a system that verifies damage claims using submitted images, a short claim conversation, user claim history and minimum image evidence requirements

The system must read `dataset/claims.csv` and produce `output.csv` with structured predictions. It must also include an `evaluation/` folder that evaluates the system on `dataset/sample_claims.csv`.

Participants may use VLMs, LLMs, structured prompting, evaluation pipelines, caching, batching, rule layers, or any other technique they prefer. The submission is judged on the quality and reproducibility of the final system, not the specific implementation style.

---

## 2. LOG FILE — LOCATION AND LIFECYCLE

The log file lives outside this repository, in the user's home directory, so it survives branch switches, worktree creation, and `git clean`.

| Platform | Path |
|---|---|
| macOS / Linux | `$HOME/hackerrank_orchestrate/log.txt` |
| Windows | `%USERPROFILE%\hackerrank_orchestrate\log.txt` |

Rules:

- Must be created if missing, including the parent directory.
- Must never be committed or added to git.
- Append-only. Never rewrite, reorder, or delete prior entries.
- Shared across all agents, sub-agents, and worktrees in this repo.
- Never log secrets. Redact API keys, tokens, cookies, private keys, and sensitive PII before writing.

---

## 3. ONBOARDING FLOW (FIRST RUN ONLY)

Run this flow only if the log file has no `AGREEMENT RECORDED:` line for the current repo root. On subsequent sessions, skip directly to §4.

### 3.1 Greeting

Open with a short, warm message. Example wording:

Welcome to HackerRank Orchestrate. You have 24 hours to design, build, and ship a system that verifies evidence for damage claims. Before we start, I need to walk you through the ground rules and get you set up. This takes about a minute.

Compute and display:

- Current system time, local timezone, ISO 8601.
- Time remaining until the challenge ends. Use the configured challenge end date if one is provided by the platform or README. If no challenge end date is present, say that the end time is not configured.
- Results announcement time, if provided by the platform or README.

If the current time is already past the challenge end, say so plainly and ask whether the user is practicing, reviewing, or re-running tests. Do not block further work.

### 3.2 Rules — recite these verbatim

1. This is a **solo** challenge. You must be the author of the submission.
2. You may use any IDE, AI assistant, or tool to help you build. The deliverable is what your system can do, not how you wrote it.
3. Your system must conform to the project contract in §6 so it can be evaluated.
4. Never commit secrets. Use environment variables and a `.env` file if needed.
5. Logging of every conversation turn to the file in §2 is mandatory and cannot be disabled.
6. Submissions are made on the HackerRank Community Platform or as otherwise instructed by HackerRank.

### 3.3 Collect the agreement

Ask the user to reply with the exact string `I agree` (case-insensitive, surrounding whitespace ignored). Do not proceed until they do.

### 3.4 Record the agreement

Append this block to the log file, then continue:

```text
## [ISO-8601 TIMESTAMP] ONBOARDING COMPLETE

AGREEMENT RECORDED: <repo_root_absolute_path>
Agent: <agent_name_or_unknown>
Language: js | ts | py | custom:<name>
System Time: <ISO-8601 local time with tz>
Time Remaining: <Xd Yh Zm, or not configured>
```

The presence of `AGREEMENT RECORDED: <this repo root>` is what future sessions check. Match the repo root exactly so agreements do not leak across unrelated clones.

---

## 4. NORMAL SESSION START (RETURNING USER)

If onboarding is already complete for this repo root:

1. Append a short `SESSION START` entry to the log (§5.1).
2. Greet the user briefly and surface the remaining time:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Sri-Ram-git/hacker-rank-orchestrate](https://github.com/Sri-Ram-git/hacker-rank-orchestrate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-02 -->
