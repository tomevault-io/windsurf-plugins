---
trigger: always_on
description: Three-tier session close-out for Claude Code — routes behavioral rules, operational knowledge, and unfinished work to three stores with three independent approval gates, plus pluggable blocking domain guards. Generic scaffold — fill in your own stores and guards. Trigger on phrases like "close out", "wrap up", "session sweep", or end-of-session signals.
---


# Tiered Close-Out

Parse the current conversation, route outputs to the right tier, and present a summary for approval before committing.

## When to Trigger

User says "close out", "wrap up", "session sweep", "tier close", or signals end of session.

## Approval Gates

This skill uses **per-tier approval** rather than a single trailing yes/no. After the summary is presented, ask the user one `AskUserQuestion` per independent decision so each tier can be accepted, narrowed, or rejected without blocking the others:

1. **Q1 — Rules-store updates** (Tier 1).
2. **Q2 — Knowledge-store ingestion** (Tier 2).
3. **Q3 — Domain guard results** (only when at least one registered guard returned `FAIL`). PASS prints a check silently; FAIL asks whether to block close-out or override with a documented reason.

Branch disposition (§5) is delegated to `superpowers:finishing-a-development-branch` and owns its own gating — do not wrap it in `AskUserQuestion`. Each tier is approved independently; never collapse them into a batch yes/no.

## Process

Scan the entire conversation and identify:

### 0. Cross-Session Context (run first)

Before analyzing the current session, look for related past sessions so the close-out can flag recurring patterns ("3rd time this thing has broken") and dedupe against existing knowledge.

```
# TODO(0): cross-session context lookup.
#
# Identify the primary topic/project of this session from the conversation,
# then query your session/knowledge backend for the top ~5 related past
# sessions. Use the results to:
#   - flag recurring patterns
#   - reference prior decisions/fixes in your knowledge entries
#   - avoid duplicate ingestion in §2
#
# Common backends:
#   - claude-mem (vector recall over prior sessions)
#   - qmd-sessions (dated markdown files, search via ripgrep)
#   - SQLite-FTS over a notes folder
#   - ripgrep over ~/notes/ or similar
#
# Implement search_prior_sessions(query: str) -> list[{date, title, snippet}].
# If the backend is unavailable, skip this step and proceed.
```

### 1. Rules — Tier 1

New behavioral instructions the user stated during the session ("always do X", "never do Y", "I want Z from now on").

```
# TODO(1): rules-store path.
#
# Default placeholder: ~/.claude/CLAUDE.md
# Replace if your project uses AGENTS.md, .cursor/rules, or a dotfile under
# the repo. The store should be a plain-text file the harness auto-loads
# every session.
RULES_STORE = "~/.claude/CLAUDE.md"
```

**Action:** Propose the edit, show the diff, gate via Q1.

### 2. Operational Knowledge — Tier 2

New facts learned during the session:

- Incidents (what broke, why, how it was fixed)
- Architectural decisions (what was decided and why)
- Patterns and anti-patterns discovered
- Business rules stated
- Infrastructure changes

```
# TODO(2): knowledge-store search + ingest.
#
# Provide two functions:
#
#   def search_existing(query: str) -> list[dict]:
#       """Return top matches for dedup. Each dict: {id, title, snippet, score}."""
#
#   def ingest_entry(payload: dict) -> None:
#       """Persist one entry. Payload: {title, body, tags: dict, source}."""
#
# Common backends:
#   - tagged markdown folder (~/notes/{domain}/{slug}.md with frontmatter,
#     search via `rg --type md`)
#   - SQLite-FTS database
#   - vector DB (Qdrant / Chroma / pgvector) with an embedding model
#   - claude-mem ingest CLI
#
# Suggested tag schema (adapt freely):
#   - domain   (which area of work)
#   - project  (which repo/product)
#   - source   (incident | architecture | policy | runbook)
```

**Action:** Build the documents, search for duplicates, show them for approval, gate via Q2.

### 3. Unfinished Work

Tasks that were started but not completed, follow-ups needed in other repos, pending decisions.

**Route to:** Summary in the close-out message. Flag cross-repo dependencies prominently. No persistent store.

### 4. Session Artifacts

Commits, PRs, design docs, scripts, branches, worktrees created during the session.

**Route to:** Summary list with full paths and links.

### 5. Branch Disposition

If the session is in a git worktree or a feature branch with commits:

**Invoke `superpowers:finishing-a-development-branch`** to present structured options:

- Merge to main (if clean, reviewed, passing)
- Create PR (if needs review or CI)
- Keep worktree (if work is ongoing)
- Remove worktree (if abandoned or merged)

This replaces ad-hoc worktree cleanup — the skill handles the full decision tree. Do **not** wrap this in `AskUserQuestion`; the downstream skill owns its own approval surface.

### 6. Cleanup

- CLI/project links that were changed during the session
- Temporary files or state

### 7. Domain Guards — pluggable blocking checks

If the session touched code/files governed by a registered guard, run the guard **before** the summary. Failures drive Q3 below.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [studio-b-ai/tiered-close-out](https://github.com/studio-b-ai/tiered-close-out) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
