---
trigger: always_on
description: A portable package of the agentic engineering protocol for AI-assisted software development. It provides a structured delegation model, risk classification, adversarial review loops, code quality gates, git workflow conventions, and named agent definitions.
---

# agentic-engineering

A portable package of the agentic engineering protocol for AI-assisted software development. It provides a structured delegation model, risk classification, adversarial review loops, code quality gates, git workflow conventions, and named agent definitions.

## Decisions
- Methodology source-of-truth lives in `content/sections/01-*.md` through `12-*.md` (P2 added `06-capability-preflight.md`; sections formerly 06-11 are now 07-12). The legacy `content/rules/agent-methodology.md` path no longer exists - edit `content/sections/` for any methodology change.
- `scripts/build-methodology.sh` is the canonical methodology assembly script. Adapter builds (`.claude/build.sh`, `.codex/build.sh`, `.gemini/build.sh`, `.kimi/build.sh`) invoke it via `bash "$REPO_DIR/scripts/build-methodology.sh"`. `.cursor/build.sh` has a different output structure and does not use it.
- `.claude/install.sh` manages `~/.claude/CLAUDE.md` via a `managed_content` Python string (lines ~364-380). Four @-import lines (METHODOLOGY.md plus 3 rules files under `rules/`) must appear in that string or rules will not auto-inject in Claude Code sessions. Re-run install.sh after any changes to that string.
- `bootstrap.sh` (repo root, PR #109) is the public `curl | bash` installer: clones to `$(pwd)/DinoStack` (override `AE_DEST_DIR`), writes the resolved path to `~/.agentic/agentic-engineering-config.json` (`repo_dir`). `/update-agentic-engineering` reads `repo_dir` at runtime (git-rev-parse validated, falls back to `~/DinoStack`) for location-aware updates.
- `skill_auto_load` opt-in enforcement (PR #97): shared hook `hooks/skill-auto-load-check.sh` wired per adapter - Claude and Codex: UserPromptSubmit; Gemini: BeforeAgent; Kimi: SessionStart. OpenCode uses a `session.created` plugin in `.opencode/plugins/session-context.ts`. Cursor has no hook (loaded via .mdc). All install scripts use read-modify-write for config updates; bare overwrite destroys existing keys. OpenCode reads from `~/.config/opencode/agentic-engineering.json`; all other adapters read from `~/.claude/agentic-engineering.json`.
- FE/QA methodology (P0+P1+P2) shipped 2026-05-28 in 14 PRs (#137-#140, #142-#145, #149-#154). `qa_criteria.method` enum is now 7 values: `browser`, `api`, `runtime-required`, `visual_conformance`, `accessibility`, `perceptual_diff`, `motion`. Capability preflight default is `blocking` as of P2 (9 agents populated; skeptic + orchestration-planner are deliberate no-ops). Canonical new reference docs: `content/references/capability-preflight.md` and `content/references/frontend-discipline.md` (7 sections + 8 Skeptic finding categories). Motion scenarios require Playwright (CDP `Emulation.setEmulatedMedia`); agent-browser cannot execute them. Storybook 6 ships URL format detection only; full SB6 framework adapter procedure improvements are deferred.
- Project-level identity override (2026-06-12): `agentic-identity --scope project` writes `<repo>/.agentic/identity.yml`; effective identity resolves by a 4-tier confirmation-first precedence (project-confirmed > global-confirmed > project-provisional > global-provisional > none); gitignored per-developer; resolver implemented in `bin/agentic-identity` + `hooks/stop-context.js` with 5 regression tests in `bin/tests/test_agentic_identity.py`.
- Telemetry-commit-on-PR (2026-06-12): the `commit_telemetry` toggle in `.agentic/config.json` (default true) makes `/implement-ticket` Phase 8 commit `.agentic/session-log/<dev>.jsonl` as a separate commit when a confirmed identity exists; `.agentic/session-log/` is git-tracked via the `!.agentic/session-log/` carve-out in `.gitignore`. Path-aware PR-checkout resolution + a HEAD-branch soft-fail guard ensure it never commits to the wrong branch. Eventual-consistency: a session's own line lands in the next ticket's Phase 8 commit; non-`/implement-ticket` PRs are not covered.

## Tools
- GitHub operations: use `gh` CLI - do not use GitHub MCP
- `gh pr create` requires an authenticated `gh` session (`gh auth status`). Run `gh auth login` if needed, then `gh pr create`.
- `rm -rf` is blocked by Claude Code permissions in this repo; remove files individually: `rm <file>` then `rmdir <dir>`
- `bin/agentic-memory` — lightweight memory retrieval tool for querying `.agentic/events.jsonl`, `MEMORY.md`, and `.agentic/context.md` on demand.

## Deploy
- Docs site: deploy steps are in `docs/technical/deploy.md` (local-only, not tracked upstream). Always verify the linked project ID before running `vercel --prod`.

## Conventions
- **Workflow for all implementation work (non-Trivial risk):**
	1. `git fetch origin` to ensure latest `main`.
	2. Spawn subagent Workers using `isolation: "worktree"` with worktrees branched from `origin/main`. Worktree path: `.agentic/worktrees/<branch-name>`.
	3. Worker implements, runs quality gates (lint, typecheck, tests), commits.
	4. Push branch to origin: `git push -u origin <branch-name>`.
	5. Open PR against `main` via `gh pr create`.
	6. Once CI/CD checks pass, auto-merge: `gh pr merge --squash --delete-branch`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Space-Dinosaurs/DinoStack](https://github.com/Space-Dinosaurs/DinoStack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
