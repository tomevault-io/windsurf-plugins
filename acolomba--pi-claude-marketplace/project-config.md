---
trigger: always_on
description: - NEVER commit to the main branch.
---

# pi-claude-marketplace

## Guidelines

### Git

- NEVER commit to the main branch.
- Branch names: `main`, `features/*`, `releases/*`. New feature branches use `features/<name>`.
- Worktrees are preferred for new feature work; create them under `.worktrees/`.
- Git commit messages: Follow the [Conventional Commits specification](https://www.conventionalcommits.org/en/v1.0.0/#specification). Titles must be at least 5 characters and no more than 72 characters. Body lines must be no more than 80 characters.
- Run `pre-commit run --all-files` (or `pre-commit run --files <changed files>`) **before** attempting `git commit`. Fix any failures, restage, and re-run until clean. Do not commit and recover from hook failures after the fact -- a failed pre-commit hook means the commit did NOT happen, so iterating with `--amend` is wrong (it would alter the previous commit).
- NEVER use `--no-verify` to skip the hooks.
- When committing from inside a worktree, prefix the commit with `SKIP=trufflehog`. The trufflehog hook's auto-updater fails to spawn child processes under the worktree sandbox even though the underlying scan succeeds; running `pre-commit run trufflehog --all-files` separately (outside `git commit`) still passes and should be done before the commit to confirm the scan is clean. Do not extend `SKIP=` to other hooks.
- When writing PR descriptions, use the `humanizer` skill if available.
- Always use `--squash` when merging PRs (`gh pr merge --squash`). The repository does not allow merge commits or rebase merges.

### Versioning

Before creating a PR, offer to bump the version in `package.json` and `sonar-project.properties`, update `package-lock.json`, and succintly record changes in `CHANGELOG.md`.

<!-- GSD:project-start source:PROJECT.md -->

## Project

`pi-claude-marketplace` is a Pi extension that gives Pi users access to Claude plugin marketplaces through a `/claude:plugin` command surface intentionally aligned with Claude Code's upstream `/plugin`. It translates Claude plugin artefacts (skills, commands, agents, MCP servers) into the equivalent Pi-native artefacts (Pi skills, Pi prompt templates, pi-subagents agents, pi-mcp-adapter MCP entries) and manages their lifecycle (install, update, uninstall, marketplace add/remove/list).

This GSD project plans a successor architecture for the V1 implementation already in this repository. The PRD at `docs/prd/pi-claude-marketplace-prd.md` (1068 lines, v1.0) is derived from the V1 source and is the authoritative specification of what the successor must deliver.

**Core Value:** A Pi user can run `/claude:plugin install <plugin>@<marketplace>` and, after `/reload`, have every supported Claude plugin component appear as a working Pi-native artefact -- atomically, recoverably, and with soft-dependency degradation that never blocks the install.

### Constraints

- **Runtime:** Node >= 20.19.0 (NFR-4)
- **Tech stack:** TypeScript strict; the resolver MUST expose discriminated `installable: true | false` so consumers cannot read `pluginRoot` from a non-installable plugin (NFR-7)
- **Pi API:** `@mariozechner/pi-coding-agent` peer dependency, currently `*` with development against `^0.70.6`; pinning a min version is a successor SHOULD (NFR-11)
- **File operations:** All disk mutations atomic (tmp + rename or atomic JSON write) -- NFR-1
- **Recovery model:** No fix may require a Pi process restart; `Run /reload` must suffice (NFR-2). All operations must be safe to retry -- idempotent or fail-clean (NFR-3)
- **Network policy:** Network is required only for GitHub-source `marketplace add` and for `update`/`marketplace update` against GitHub-source marketplaces; `install`, `list`, `uninstall`, `marketplace remove`, and path-source `marketplace add` MUST NOT touch the network (NFR-5)
- **Containment:** Refuse to write outside `<scopeRoot>/pi-claude-marketplace/`, `<scopeRoot>/agents/`, `<scopeRoot>/mcp.json`, `<scopeRoot>/claude-plugins.json`, or `<scopeRoot>/claude-plugins.local.json` (NFR-10)
- **Quality bar:** `npm run check` must stay green -- typecheck + ESLint + Prettier + tests (NFR-6)
- **Output channel:** All user-visible messages MUST go through `ctx.ui.notify(message, severity)`; direct `process.stdout`/`process.stderr` writes forbidden in command/bridge code (IL-2). Single sanctioned `console.warn` is the load-time legacy migration save failure (IL-3)
- **No telemetry V1:** No metrics, no event sink, no analytics endpoint (IL-4)
- **English only V1:** No message catalog, no locale negotiation (IL-1)
- **Scope model:** Exactly two scopes -- `user` (Pi agent dir; defaults to `~/.pi/agent/` and honors `PI_CODING_AGENT_DIR`) and `project` (`<cwd>/.pi/`). Claude Code's `local` scope is not introduced (SC-1)

<!-- GSD:project-end -->

<!-- GSD:stack-start source:research/STACK.md -->

## Technology Stack

## Executive Verdict

## Recommended Stack

### Core Technologies


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [acolomba/pi-claude-marketplace](https://github.com/acolomba/pi-claude-marketplace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
