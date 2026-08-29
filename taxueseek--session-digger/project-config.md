---
trigger: always_on
description: Session Digger mines Claude Code session JSONL files for insights. It helps users recall past work, trace decisions, learn from mistakes, and understand file histories by analyzing `~/.claude/projects/` conversation data.
---

# Session Digger — Plugin Development Guide

## Purpose

Session Digger mines Claude Code session JSONL files for insights. It helps users recall past work, trace decisions, learn from mistakes, and understand file histories by analyzing `~/.claude/projects/` conversation data.

## Architecture

```
commands/  → User-facing slash commands (entry points)
agents/    → Task-specific agents dispatched by commands via the Task tool
skills/    → Reusable knowledge (parsing rules, git patterns, synthesis taxonomy)
scripts/   → Python/bash tools that do the actual JSONL parsing and extraction
```

**Flow:** Commands dispatch to agents. Agents use skills for domain knowledge and call scripts (via Bash tool) for data extraction. Scripts are thin bash wrappers around the `scripts/echolib/` package.

### Commands
- `/recall` — Search and analyze past sessions (auto-uses FTS index after `/index`)
- `/recap` — Summarize recent sessions
- `/timeline` — Chronological project history (sessions + git)
- `/lessons` — Extract lessons learned
- `/dashboard` — Global memory overview and staleness alerts
- `/audit` — Memory staleness audit (heuristic or deep)
- `/extract` — Extract knowledge from conversation sessions
- `/prune` — Interactive memory cleanup
- `/analyze` — Analyze sessions for patterns (retry loops, errors, corrections)
- `/apply` — Interactive rule approval and persistence (y/n/e/a/q review)
- `/topics` — Topic segmentation and boundary detection
- `/index` — Build/update SQLite FTS search index
- `/import` — Import external conversations (WeChat, JSON, CSV, transcript)
- `/save-summary` — Save analysis result as cached summary for future recall
- `/trend` — Longitudinal trend analysis (week/month-over-month, by-theme, regressions)
- `/optimize` — Cross-session skill gap analysis → SKILL.md improvement proposals

### Agents
- `recall` — Unified search: session finding, decision archaeology, mistake hunting
- `file-historian` — Trace a file's history across sessions and git
- `analyze` — Deep analysis of specific sessions
- `schema-scout` — Detect JSONL schema changes
- `memory-auditor` — Deep content-aware memory verification

### Skills
- `jsonl-core` — Canonical JSONL parsing infrastructure and record type reference
- `git-mining` — Git log/blame/diff patterns for correlating commits with sessions
- `experience-synthesis` — Taxonomy for categorizing insights (decisions, mistakes, patterns)
- `memory-management` — Memory format, staleness scoring, and routing knowledge

### Scripts
All in `scripts/`, require only Python 3.6+ (stdlib only) and bash. Git scripts additionally require git.

**Core library:**
- `echolib/` — Core Python parsing package (no pip dependencies)

**Unified engine (v0.7):**
- `sd-recall.py` — Unified single-process recall engine. Replaces bash pipeline (list-sessions + extract-messages + extract-tools). Uses SQLite FTS index when available, falls back to file scan. Supports `search`, `sessions`, `stats` subcommands.
- `index-builder.py` — Build/update SQLite FTS5 index. v1.1: now stores rich stats (tool_usage_json, tool_errors_json, flags_json, duration_seconds, project_name, tags, outcome) for trend/skill-gap analysis. Subcommands: `build` (incremental), `search` (FTS query), `detail` (session metadata), `stats` (index overview).
- `dialog-adapter.py` — Import external conversation formats (WeChat export, generic JSON/CSV, plaintext transcripts) into session-digger's JSONL schema. Auto-detects format.
- `topic-segmenter.py` — Topic boundary detection via time-gap + content-similarity heuristics. Outputs labeled segments with keywords.

**Trend & skill-gap analysis (v0.8):**
- `trend-engine.py` — Longitudinal analysis over the SQLite index. Three modes: `period-over-period` (week/month comparison with deltas), `by-theme` (aggregate by project/tag/agent), `regressions` (detect tools getting worse). Reads ONLY the index, never raw transcripts.
- `skill-gap-finder.py` — Mine the index for recurring pain points (tool errors, retry loops, long conversations, project outliers) and draft reviewable SKILL.md proposals. Matches patterns to installed skills via keyword overlap. Never auto-edits skill files.
- `format-detector.py` — Signature-matching format detection for unknown agent transcripts. Scores known formats (Claude Code, Grok, Kimi Code, Cline, Aider, generic markdown) and surfaces unknown formats with sample keys for extension.

**Per-command scripts:**
- `analyze-session.sh` — Pattern analysis: retry loops, errors, user corrections. Invoked by `/analyze`.
- `apply-rules.sh` — Format, review, and persist candidate rules. Invoked by `/apply`.
- `post-compaction-hook.sh` — Claude Code PreToolUse hook: detect compaction and remind about context recovery tools.

**Group chat profiles (v0.7.1):**
- `chat-profiles.py` — Incremental participant profile extraction for group chats.
  Uses append-only merge rules inspired by baoyu-wechat-summary: quotes/events
  append unbounded, tags/interests merge with frequency sorting, speaking style
  only refines at 100-message thresholds. Outputs one JSON per participant.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [taxueseek/session-digger](https://github.com/taxueseek/session-digger) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
