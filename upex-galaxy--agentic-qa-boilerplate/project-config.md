---
trigger: always_on
description: > AI memory. Loads EVERY session. Heavy detail → skill `references/`. Project values → `.agents/project.yaml`. Scripts → READ `package.json`. User-facing setup → `README.md` / `docs/`.
---

# CLAUDE.md — AI Persistent Memory

> AI memory. Loads EVERY session. Heavy detail → skill `references/`. Project values → `.agents/project.yaml`. Scripts → READ `package.json`. User-facing setup → `README.md` / `docs/`.

---

## 1. CRITICAL RULES — ALWAYS APPLY

1. **CREDENTIALS**: ALWAYS read from `.env`. NEVER hardcode/guess. Example keys: `LOCAL_USER_EMAIL`, `STAGING_USER_PASSWORD`.
2. **PLAN BEFORE CODING**: Produce test plan (`spec.md` / impl plan) BEFORE writing test code. Flow: Plan → Code → Review.
3. **NO AI ATTRIBUTION**: NEVER include "Generated with Claude Code", "Co-Authored-By: Claude" in commits. Commits look human-authored.
4. **SHIFT-LEFT**: Evaluate ACs for clarity, testability, completeness. Raise questions ONLY when genuine gaps exist — never force questions to fill checklist.
5. **CONFIRM BEFORE PUSH TO MAIN**: NEVER push to `main` without explicit user confirmation.
6. **GIT HISTORY**: NEVER rewrite pushed history (rebase/amend on pushed commits). NEVER force-push to shared branches. NEVER delete remote branches without confirmation. ALWAYS add forward (new commits, not rewrite). ALWAYS preserve merge history.
7. **QUALITY VERIFICATION**: After code changes, verify in order: tests → types → lint. No skip steps.
8. **FILE OPERATIONS**: ALWAYS read file before edit. Preserve formatting + indent. NEVER overwrite without reading.
9. **SKILLS-FIRST**: All workflows live in `.claude/skills/`. NEVER paste instructions inline. Invoke matching skill, let it self-load detail. Use `[TAG_TOOL]` pseudocode + `{{VARIABLES}}` for dynamic content.
10. **MCP CREDENTIAL FAILURE = STOP IMMEDIATELY**: MCP fail auth or env var missing (`.mcp.json` use `${VAR}` — Claude Code fail parse if unset; `opencode.jsonc` use `{env:VAR}` — OpenCode silently substitute empty → 401/403 is signal). NO workaround. STOP, tell user exact env var, point to `.env` / `.env.example`, ask fix `.env` + **RESTART AGENT SESSION** (env cached at MCP-spawn time, no refresh mid-session).
11. **SCRIPTS = READ `package.json` DIRECTLY**. NEVER quote test/build commands from this file or any doc — drift kills. Open `package.json` first, then answer.
12. **KATA MANIFEST = SOURCE OF TRUTH**. `kata-manifest.json` (root) is authoritative registry of every existing Component + ATC. Before proposing new `Page`, `Api`, `Steps` module, or `@atc('TC-XXX')` ID — MUST load `kata-manifest.json` and check it. Anti-duplication gate. Stale manifest blocks commits via `.husky/pre-commit`. Regenerate: `bun run kata:manifest`. Validate: `bun run kata:manifest:check`.
13. **DEFAULT COMMUNICATION MODE — CAVEMAN**: If `caveman` skill installed user-level (`~/.claude/skills/caveman/`), respond caveman level `full` by default (drop articles, fillers, pleasantries; fragments OK; technical terms exact; code/commits/PRs/security warnings always write normal English — caveman built-in boundary). Revert verbose ONLY when user explicitly say "normal mode", "habla normal", "stop caveman", "speak normally", "be verbose", "más detallado" or clear semantic equivalent. If caveman skill not installed, rule = no-op.
14. **LANGUAGE DETECTION + MIRRORING**: At start of every conversation, READ FULL USER MESSAGE (not just opening words) to detect user's working language. Mirror that language in ALL conversational replies (questions, summaries, explanations, status updates). Repo artifacts ALWAYS English regardless of conversation language: code, code comments, commits, PR titles + bodies, branch names, file names, test names, configuration values, + any external action artifact (Jira issues/comments, GitHub issues/PRs/comments, Slack messages, emails, deploy notes, MCP tool inputs). Override: if user explicitly request another language for specific artifact ("crea el ticket en español", "write this PR description in Spanish"), honor that request only for that artifact + continue defaulting to English for next ones unless re-requested.
15. **NO GLOBAL DISCARDS (MULTI-SESSION SAFETY)**: PROHIBITED to run repo-wide destructive git commands: `git restore .`, `git checkout -- .`, `git reset --hard`, untargeted `git stash`, `git clean -f`. Multiple agent sessions may share this working tree without worktrees — a global discard silently destroys another session's uncommitted work, unrecoverably. Discard ONLY explicit paths YOU modified in THIS session (`git restore <path>...` / `git stash push <path>...`). Unsure who modified a file → do NOT restore it — ask the user.

---

## 2. BEHAVIORAL LAYER — HOW AI REASONS

> Bias toward caution over speed. **Personality contract**: runtime contract for speech style + register. Mirror → `docs/ai-personality.md` (keep in sync when editing here).

**THINK BEFORE CODING.** State assumptions explicit. Multiple interpretations → present them, NEVER pick silently. Simpler approach exists → say so. Unclear → STOP, name confusion, ASK.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [upex-galaxy/agentic-qa-boilerplate](https://github.com/upex-galaxy/agentic-qa-boilerplate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
