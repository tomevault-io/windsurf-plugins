---
trigger: always_on
description: Use when asked for a *"deep parallel scan"*, *"full scan loop with all agents"*, or to run the convergence loop across subagents. This trades the single-context guarantee for parallelism: each lens gets its own convergence loop, and coverage/ledger state is reconciled at merge time. Note each lens subagent independently reads every in-scope line, so total read cost scales with the number of lenses.
---

# SAST Security Assessment

Your goal is to identify security vulnerabilities in the codebase located in the current working directory by orchestrating the `llm-sast-scanner` skill across parallel subagents — one per vulnerability lens — so each lens runs in its own isolated context.

All output is written to a `.llm-sast-scanner-cache/` folder in the project root. Steps whose output file already exists **and is marked complete (the file ends with the `<!-- LLM-SAST-COMPLETE -->` sentinel)** are skipped, so this is safe to re-run after fixing issues. A file that exists **but lacks the terminal sentinel is from a crashed/partial step** and is re-run and overwritten — existence alone is never proof of completion.

> **Skill resolution:** subagents invoke skills by name (`llm-sast-scanner`, `llm-sast-scanner-full-scan-loop`). Each tool loads them from its own skills directory — Claude Code from `.claude/skills/`, Cursor/Codex/agents from `.agents/skills/`. Both directories are symlinks to the single canonical skill source at the repo root, so the two runtimes always run identical skill content.

---

## Arguments

This orchestrator forwards optional tagged arguments to the underlying skill.

- `adv=critical,high,medium` (case-insensitive, comma-separated) — controls which severities go through the scanner's **Step 6: Adversarial Impact Validation**. When omitted, Step 6 is skipped. Pass the same `adv=` value to every Step 2 subagent and to the Step 3 report agent.

---

## Step 1: Codebase Analysis & Threat Modeling

Check if `.llm-sast-scanner-cache/architecture-threat-model.md` already exists **and is current** — reuse it only when `project-memory.md`'s `last-scanned-sha` equals `git rev-parse HEAD` (stack provably unchanged). If it exists and is current, skip the analysis below (but still run the project-memory step at the end of this section). If the SHA differs (or is `unknown`, or the file is missing), **regenerate** it — the code changed, so entry points / detected stack / the stack-gated allowlist may have too, and a stale `architecture-threat-model.md` silently drops newly-applicable lenses.

Otherwise, **in-session** (not as a subagent, since later steps read its output), run the `llm-sast-scanner` skill's **Step 1 (Understand Scope)** over the whole repo and write a short architecture/threat-model brief to `.llm-sast-scanner-cache/architecture-threat-model.md` covering: languages & frameworks, entry points (routes/handlers/CLI/jobs), trust boundaries, authN/authZ model, data stores, outbound calls, and the **detected stack** so later lenses can skip inapplicable reference files. Also record the **per-lens stack-gated reference allowlist** derived from the files actually present (gateable platform/language/infra references whose signals appear, plus the always-loaded language-agnostic classes), so lenses share one definition of applicable classes and drop only provably-absent stacks.

**Project memory (always, even when `architecture-threat-model.md` already existed):** ensure `.llm-sast-scanner-cache/project-memory.md` exists; if absent, initialize it from the template in the base skill's **Project Memory Protocol**. This file carries cross-scan hints (confirmed findings, confirmed false-positive patterns, project security primitives, hotspots) and is consumed by every detection subagent as *hints, never authority*. Also add `.llm-sast-scanner-cache/` to the repo's `.gitignore` if not already ignored.

**Wait for this step to finish before proceeding.**

---

## Step 2: Vulnerability Detection (Parallel)

Start **one subagent per lens**, all **in parallel**. Skip any lens whose results file already exists **and ends with the `<!-- LLM-SAST-COMPLETE -->` sentinel**; a present-but-unmarked file is a crashed/partial run — re-run that lens and overwrite it.

Give each subagent the same instruction pattern, substituting the lens name, class list, and results path from the table below:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cujanovic/llm-sast-scanner](https://github.com/cujanovic/llm-sast-scanner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-12 -->
