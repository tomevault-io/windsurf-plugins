---
trigger: always_on
description: Temper reference: review
---



# Review: Confidence-Scored Code Review

**Goal:** Review recent changes with high signal-to-noise ratio. Parallel subagent review, confidence scoring, review memory, and intent validation.

## Active Skills

- **Context Engineering** — load hierarchical context at stage start (rules → arch → source → errors, under 2K lines/task)
- **Temper Core** — stack detection, pack resolution, quality gates

## Prerequisites

**DO NOT RUN if:**

- Code does not compile
- Tests are failing
- Build is broken

**RUN ONLY AFTER:**

- Build succeeds
- All tests pass
- Or: auto-chained from /temper:build (which already validated)

For confidence scoring and review memory, apply the temper-core skill.

## Execution

### Context Loading

This stage may run in two modes:
- **Standalone** (`/temper:review`) — runs in current context, handles its own gate
- **Agent subprocess** (from `/temper`) — starts with CLEAN context, only loads what's listed below

**Subprocess mode override:** When running as an Agent subprocess, do NOT show AskUserQuestion gates or clear context. Return the review summary to the orchestrator. The orchestrator handles all gate decisions and context transitions.

In both modes, the review methodology is identical.

Files to load at start:
1. Run `git diff --name-only` to identify changed files
2. `$CLAUDE_PLUGIN_ROOT/.claude-plugin/reference/review.md` (this file)
3. `.temper/specs/{feature}/intent.md` (for intent validation, if exists)
4. `.temper/specs/{feature}/build-context.json` (if exists — build deviations and test results)

### Step 1: Gather Context

```bash
# 1. Get changed files
git diff --name-only HEAD~1..HEAD  # if committed
git diff --name-only               # if uncommitted

# 2. Get diff statistics
git diff --stat HEAD

# 3. Read temper.config for review settings
# - block-on: which severities block
# - confidence-threshold: minimum confidence to show
# - auto-fix: whether to auto-fix

# 4. Read active pack rules
# - Load enabled packs from .claude/packs/
# - Load stack-specific rules from .claude/packs/stacks/{detected-stack}.md

# 5. Read review memory
# - Load .temper/review-memory.json if exists
# - Contains: dismissed patterns, accepted patterns, auto-rules

# 6. Find active intent.md
# - If chained from /temper:build: use the same spec (build context contains: spec name, feature path)
# - If single spec in .temper/specs/: use that intent.md
# - If multiple specs: check git branch name for match, or ask user which spec to review
# - If no specs: skip intent validation (existing behavior)
```

### Step 1.5: Diff-Aware Fingerprinting

Before launching subagents, build a diff fingerprint that classifies each changed region by risk level. This focuses review energy where it matters most.

```
1. Extract unified diff with context:
   git diff -U5 HEAD~1..HEAD  # if committed
   git diff -U5               # if uncommitted

2. For each changed file, classify the change:
   a. Change type:
      - ADDITION: New file (git status shows "??")
      - MODIFICATION: Existing file with hunks
      - DELETION: File removed
      - RENAME: File moved (git status shows "RNN")

   b. For MODIFICATION files, parse each hunk:
      - Identify the function/method containing the change
        (parse upward from hunk for def, function, class, const, etc.)
      - Classify the change:
        LOGIC — business logic, conditionals, calculations
        STRUCTURE — new class, new method, refactored signature
        CONFIG — settings, environment, feature flags
        TEST — test files, test helpers, fixtures
        IMPORT — import/require changes only

   c. Detect risk signals per hunk:
      - SECURITY: password, token, jwt, encrypt, decrypt, hash, auth,
        secret, credential, api-key, session
      - DATA_MUTATION: insert, update, delete, create, drop, alter,
        save, persist, remove
      - ERROR_HANDLING: throw, catch, error, exception, reject, fail
      - CONCURRENCY: async, await, promise, spawn, thread, goroutine,
        channel, mutex, lock
      - EXTERNAL_API: fetch, http, request, client, axios, curl, grpc

3. Build the fingerprint (ephemeral — not persisted):

   DIFF FINGERPRINT:
     Files: {N} changed ({A} additions, {M} modifications, {D} deletions)
     Hunks: {N} total ({L} logic, {S} structure, {C} config, {T} test, {I} import)
     High-risk regions: {N}
       - {file}:{hunk} — {risk signals}
       - {file}:{hunk} — {risk signals}
     Security sensitivity: {N} CRITICAL, {N} HIGH, {N} MEDIUM, {N} LOW
```

Pass this fingerprint to all subagents in Step 2. Subagents must:
- Focus 80% of attention on hunks with risk signals
- Review remaining changed lines at standard depth
- Include fingerprint summary in their findings

### Step 2: Launch Parallel Review Subagents

**If changed files span multiple domains (e.g., backend + frontend), launch parallel subagents.**

Each subagent receives:

```
Review the following files for issues. For each issue found, provide:
1. Severity: CRITICAL / HIGH / MEDIUM / LOW
2. Confidence: 0.0-1.0 (how certain you are this is a real issue)
3. Category: logic / security / performance / quality / standards / architecture / test-gap
4. Location: file:line
5. Description: what the issue is
6. Suggestion: how to fix it

Rules to enforce:
{content of active pack rules}


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [galando/temper](https://github.com/galando/temper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
