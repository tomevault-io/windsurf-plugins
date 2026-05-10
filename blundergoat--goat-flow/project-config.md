---
trigger: always_on
description: Documentation framework for AI coding agent workflows. Markdown docs + Bash scripts + TypeScript CLI auditor.
---

# Copilot Instructions - v1.5.0 (2026-05-04)
Documentation framework for AI coding agent workflows. Markdown docs + Bash scripts + TypeScript CLI auditor.

goat-flow is a harness — guardrails, memory, and workflows for AI coding agents. Five concerns drive every design decision: **Context** (what you read), **Constraints** (what you may never do), **Verification** (how work is checked), **Recovery** (how state survives failure), **Feedback loop** (how mistakes become permanent fixes).

This repo is the goat-flow controlling workspace. When the dashboard or CLI operates on a selected target project, commands like `audit` and `quality` run against that target - not this repo. Keep the two contexts separate: framework code lives here, project-specific harness content lives in the target.

## Truth Order

User instruction > `.github/copilot-instructions.md` > `.goat-flow/architecture.md` > on-demand skills/templates.

## Autonomy Tiers

**Always:** Read any file, lint scripts, edit within assigned scope. Session logs at `.goat-flow/logs/sessions/` are OPTIONAL continuity notes - write one when `/compact` fires without an active milestone file, otherwise skip. Learning-loop updates (lessons/footguns/decisions) follow the conditional rules above: update only when VERIFY caught a failure or you corrected course.

**Ask First** - before proceeding, state: boundary touched, related code read (yes/no), footgun entry checked (or "none"), local instruction checked, rollback command.

Boundaries: instruction files (`.github/copilot-instructions.md`, `CLAUDE.md`, `AGENTS.md`, `GEMINI.md`); workflow/manifest (`workflow/setup/`, `workflow/skills/`, `workflow/manifest.json`); architecture/playbooks (`.goat-flow/architecture.md`, `.goat-flow/skill-reference/`); server runtime (`src/cli/server/terminal.ts`, `src/cli/server/dashboard.ts`); agent configs (`.claude/**`, `.codex/**`, `.gemini/**`, `.agents/**`); CI/hooks (`.github/workflows/**`, `.github/actions/**`, `.github/hooks/**`, `.github/skills/**`); any add/remove/rename; changes spanning 3+ docs.

**Never:** If interrupted or told no changes, freeze writes; run only read-only status/diff checks until the user explicitly asks for cleanup, revert, or apply. Delete docs without replacement. Modify .env/secrets. Push. Commit unless asked. Invent hypothetical examples. Overwrite existing files without checking destination (`ls` before `mv`/`cp`/Write; use `mv -n`). Delete/move/overwrite 5+ files in one operation without listing targets and getting confirmation.

## Hard Rules
- If file exists, modify in-place. NEVER create `_modified`, `_new`, `_backup`, `_v2` variants.
- Severity: SECURITY > CORRECTNESS > INTEGRATION > PERFORMANCE > STYLE.
- MUST maintain cross-file consistency: same concept, same description everywhere.
- MUST preserve file-level evidence in footguns and examples. Use grep-friendly semantic anchors (function name, unique string, `(search: "pattern")`), not line numbers - they go stale on every edit (per ADR-024).
- MUST use real incidents, never hypothetical. `.goat-flow/architecture.md` is canonical source of truth.
- Sub-agents: ONE objective, structured return (paths, evidence, confidence, next step), 5-call budget. Blocked → one question with recommended default.
- No features, abstractions, or error handling beyond what was asked. Gold-plating is scope creep.
- Ambiguous requirements: present interpretations, don't pick silently.
- Use current Copilot CLI commands (`/agent`, `/review`, `/research`, `/tasks`) when appropriate; use `/fleet` only for explicit or genuinely independent parallel work.
- Treat `.github/actions/**`, `.github/hooks/hooks.json`, `.github/hooks/deny-dangerous.sh`, `.github/skills/**`, `.github/copilot-instructions.md`, and `.copilotignore` as security-sensitive runtime surfaces; verify after touching them.
- `.github/agents/` is intentionally out of scope; CI/CD, hooks, prompts, or skills work should prefer `goat-security` or `goat-review`.

## Key Resources

- **Learning loop** (grep before every change): `.goat-flow/footguns/`, `.goat-flow/lessons/`, `.goat-flow/patterns/`, `.goat-flow/decisions/`
- **Tool playbooks**: `.goat-flow/skill-reference/browser-use.md`, `.goat-flow/skill-reference/page-capture.md` — read BEFORE declaring a tool unavailable

## Essential Commands

```bash
shellcheck scripts/*.sh scripts/maintenance/*.sh
bash -n scripts/*.sh scripts/maintenance/*.sh
npm run typecheck
npm test
```

Situational: `preflight-checks.sh` (full gate), `bump-version.sh <ver>` (release), `test:full` (pre-release), `stats . --check` (learning-loop), `.github/hooks/deny-dangerous.sh --self-test` (hook check).

## Execution Loop: READ → SCOPE → ACT → VERIFY

When a goat-* skill is active, its Step 0 replaces READ and selects the skill's mode/depth. SCOPE still applies before writes: a skill may write when its selected mode permits writes or the user explicitly approves them. `/goat-plan` File-Write may create gitignored milestone files without a separate approval gate; `/goat-debug` D3 still requires approval before fixes. Resume at ACT after Step 0 output or when a blocking gate releases.

### READ

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [blundergoat/goat-flow](https://github.com/blundergoat/goat-flow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-10 -->
