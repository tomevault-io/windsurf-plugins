---
trigger: always_on
description: This file is the single source of truth for AI coding agents working in this repository.
---

# AGENTS.md

This file is the single source of truth for AI coding agents working in this repository.

For task-specific workflow details, use the repo-local skills in `.agents/skills/abo-*`.

## Project Summary

Audiobook Organizer is a Go application for organizing and renaming audiobook libraries using metadata from `metadata.json` files and embedded metadata in EPUB, MP3, and M4B files.

The repository supports these user-facing entrypoints:

- `audiobook-organizer` for non-interactive CLI organization
- `audiobook-organizer tui` for interactive terminal organization
- `audiobook-organizer rename` for CLI renaming
- `audiobook-organizer rename-tui` for interactive terminal renaming
- `audiobook-organizer web` for the local browser UI
- `audiobook-organizer gui` as a compatibility alias for the local browser UI

The project ships one `audiobook-organizer` binary with CLI, TUI, rename, Audiobookshelf, and local web UI support.

## Repository Shape

- `main.go`: CLI entrypoint
- `cmd/`: Cobra commands and top-level flag/config wiring
- `internal/organizer/`: core organization and rename logic
- `internal/tui/`: Bubble Tea TUI flows and models
- `internal/app/`: application service layer used by the web API
- `internal/server/`: local HTTP server, token checks, JSON routes, and embedded static assets
- `web/`: Vue/Vite frontend for the local browser UI
- `docs/`: user-facing documentation
- `testdata/`: test fixtures for audio and metadata scenarios
- `internal/organizer/integration/`: integration tests
- `test/abs/`: Audiobookshelf test harness and E2E tests

The supported UI is the local browser UI through `audiobook-organizer web`, `audiobook-organizer gui`, `cmd/web.go`, `cmd/gui.go`, `internal/server/`, `internal/app/`, and `web/`.

## Repo-Local Skills

Use these skills for repeatable Audiobook Organizer workflows:

- `$abo-workflow`: route broad maintainer requests to the right specialist skill.
- `$abo-feature`: implement focused features across CLI, core, TUI, web, or ABS boundaries.
- `$abo-bugfix`: reproduce, fix, and verify regressions with focused tests.
- `$abo-issue-create`: create or reuse an issue and prepare the issue branch.
- `$abo-issue-watcher`: inspect issue status, comments, linked PRs, and next steps.
- `$abo-issue-verify`: verify acceptance criteria, tests, docs, changelog, and ABS matrix obligations.
- `$abo-issue-closeout`: finish issue hygiene and close only when appropriate.
- `$abo-tests`: select, write, and run repo-native Go, TUI, server/app, web, and docs checks.
- `$abo-abs-tests`: handle Audiobookshelf harness, ABS E2E, and `test/abs/test-matrix.md` work.
- `$abo-web-ui`: work only on the current local browser UI design in `web/`, `internal/server/`, `internal/app`, `cmd/web.go`, and `cmd/gui.go`.
- `$abo-audit`: audit Go and current web UI dependencies without changing files.
- `$abo-updater`: update Go and current web UI dependencies, then verify.
- `$abo-docs`: maintain docs, AGENTS.md, changelog, and repo-local skill references.
- `$abo-pr`: route PR drafting, creation, watching, and closeout.
- `$abo-pr-writer`: draft or update PR descriptions.
- `$abo-pr-create`: commit, push, and create a PR into protected `master`.
- `$abo-pr-watcher`: watch PR CI, review comments, issue comments, and branch freshness.

Shared skill references live in `references/abo-assistant/`. Keep AGENTS.md focused on durable repo rules; put detailed repeatable procedures in the relevant skill or shared reference.

## GitHub Workflow

- Track non-trivial code and documentation changes with a GitHub issue before editing files.
- If an issue already exists, use it. If not, create one with the goal, motivation, and acceptance criteria.
- Create a dedicated branch from `master` for each issue before editing files. Start from a fresh remote base with `git fetch origin master` and `git switch -c <branch> origin/master`.
- Use descriptive branch prefixes by work type: `feature/<short-name>` for features, `fix/<short-name>` for bug fixes, `docs/<short-name>` for documentation-only changes, and `chore/<short-name>` for maintainer/tooling work.
- Verify the active branch with `git status --short --branch` before editing, committing, or pushing. Do not commit or push from `master` except for explicitly approved tiny edits or explicit repository maintenance such as an approved history rewrite.
- `master` is protected. Normal work must merge through a pull request with required checks passing. Repository auto-merge is enabled for the single-maintainer workflow, so do not require a separate approval unless branch protection is intentionally changed. Admin enforcement is enabled; do not bypass protection for normal work.
- Keep the issue updated while working. Add comments for scope changes, important implementation decisions, blockers, test results, and follow-up work discovered during implementation.
- Keep commits focused on the issue. Do not mix unrelated cleanup, refactors, or separate features into the same branch.
- As part of each feature or fix, decide whether tests, docs, and `CHANGELOG.md` need updates. If they do, include them in the same branch. If they do not, note why in the PR.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jeeftor/audiobook-organizer](https://github.com/jeeftor/audiobook-organizer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
