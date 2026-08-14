---
trigger: always_on
description: - `projmux` is a standalone tmux session-management application.
---

# Agent Guide

## Scope
- `projmux` is a standalone tmux session-management application.
- Keep portable session-management behavior in `projmux`.
- Keep machine-local policy outside the application unless the migration plan explicitly calls for it.
- Keep `AGENTS.md` focused on repo contract. Personal agent recipes, reverse-engineering notes, and machine-local operating memos belong in external local docs (Obsidian/dotfiles anchors), not this tracked file.
- Local-only agent overlays may live in an untracked `AGENTS.local.md` anchored via dotfiles. This tracked file stays shareable and tool-agnostic.

## Startup Checks
- Confirm you are in the intended checkout/worktree with `pwd`.
- Check local state with `git status --short`.

## Branch And Checkout Rules
- Use one branch per task. Preferred names: `feat/<topic>`, `fix/<topic>`, `docs/<topic>`, `refactor/<topic>`, `chore/<topic>`.
- Use a dedicated checkout/worktree per task when parallel work would otherwise collide.
- Keep one agent per checkout/worktree. Do not share a dirty checkout across agents.
- If another agent owns a file, do not overwrite their changes. Adjust around them or coordinate a handoff.
- Keep changes narrow. Split docs, bootstrap, migration, and feature work into separate branches unless they are inseparable.

## Branch Protection And PR Flow
- `main` is protected by the repository ruleset `main-protect`. Direct pushes to `main` are blocked even for repository admins.
- Every change ships through a pull request. The required status check is the CI `Test` job.
- Admin bypass mode is `pull_request`: the admin can self-merge a PR without approvals, but the PR itself is mandatory.
- Default merge method is **squash**. The PR title becomes the squash commit subject and is what release-please parses, so write it as a Conventional Commit. Follow [docs/pr-guideline.md](docs/pr-guideline.md) for full conventions.
- `make install` cannot run before the PR is merged into `main`. The full team-lead loop is: push branch → open PR → wait for CI → merge → `git pull --ff-only` → `make install`.

## Standard Dev Flow
- Make targets are the contract for local validation. Keep them stable and predictable.
- Run work in this order before opening a PR:
  1. `make fmt`
  2. `make fix`
  3. `make test`
  4. `make test-integration`
  5. `make test-e2e`
- Then push and open the PR:
  6. `git push -u origin <branch>`
  7. `gh pr create --title ... --body ...` (Conventional Commit title; see [docs/pr-guideline.md](docs/pr-guideline.md)).
  8. Wait for the `Test` check to turn green (`gh pr checks <num> --watch`). Use `--auto` on `gh pr merge` if you want it queued.
  9. `gh pr merge <num> --squash --delete-branch`.
- Promote the build only after merge:
  10. `git -C <repo> pull --ff-only`
  11. `make install` — atomic replace of `$(go env GOPATH)/bin/projmux` plus `projmux tmux apply`. **Never run it before step 10**; pre-merge state has not cleared CI yet and may not match what `main` will hold.
  12. Retire the merged checkout/worktree with your local tooling if you used one.
- If a target is missing for the area you are changing, add it or leave the repository in a state where the gap is explicit in docs and review notes.
- If behavior changes, update the maintained test list in [docs/agent-workflow.md](docs/agent-workflow.md) in the same branch.
- Do not skip `fmt` or `fix` because tests passed. Formatting, automatic fixes, and test execution are separate gates.

## Hook Contract Stability
- The post-create hook contract (path `~/.config/projmux/hooks/post-create`, `PROJMUX_*` env vars, 5s timeout) is part of the public API. Adding, removing, or renaming any `PROJMUX_*` env var, or moving the hook path, requires at minimum a minor version bump.

## Release Flow
- `release-please-action` watches `main`, accumulates Conventional Commit subjects, and opens or refreshes a "chore(main): release X.Y.Z" PR. That PR contains the version bump (`internal/version/version.go` + `.release-please-manifest.json`), `CHANGELOG.md` updates, and the release notes.
- Merging the release PR pushes the `vX.Y.Z` tag and creates the GitHub Release with auto-generated notes.
- `.github/workflows/release.yml` triggers on the tag push, builds the linux/darwin × amd64/arm64 matrix, and uploads tarballs to the release that already exists (`gh release upload --clobber`). Do not add hardcoded notes back to that workflow — release-please owns the notes.
- Non-Conventional commit subjects on `main` are silently skipped by release-please. Keep PR titles strict; squash merge ensures the PR title is the only subject that lands.

## Configuration And Environment

Project root and discovery:

- `PROJMUX_PROJDIR` is the canonical project-root env. It accepts an OS-native PATH-style multi-value (`filepath.SplitList`): the first non-empty entry is the primary repo root (memoized to `~/.config/projmux/projdir`), and any additional entries are prepended to managed roots. The legacy `PROJDIR`/`RP` env vars are no longer honored.
- `PROJMUX_MANAGED_ROOTS` is the colon-separated search-root override (priority: env > saved file > defaults). Legacy alias `TMUX_SESSIONIZER_ROOTS` is still honored at runtime.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [crevissepartners/projmux](https://github.com/crevissepartners/projmux) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
