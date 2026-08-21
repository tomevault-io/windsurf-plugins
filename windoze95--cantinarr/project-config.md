---
trigger: always_on
description: Operating manual for AI agents and human contributors. `CLAUDE.md` imports this file — this is the single canonical copy; edit it here, never there.
---

# Agent Instructions

Operating manual for AI agents and human contributors. `CLAUDE.md` imports this file — this is the single canonical copy; edit it here, never there.

## Collaboration

- Do not just agree with the user by default. If a request would weaken the project, hurt maintainability, reduce correctness, or make the task outcome worse, push back clearly and suggest the better path.

## Git workflow

- Before starting any PR-sized change: `git fetch origin`, make sure local `main` is even with `origin/main`, and create a feature branch from that fresh base.
- Branch names follow `feat/…`, `fix/…`, `docs/…`, `chore/…`.
- Do not open PRs directly from `main`. If work accidentally happens on `main`, verify `main` is still even with `origin/main`, then move the work to a feature branch before committing.
- Preserve user work. Do not revert or delete unrelated local changes or untracked files.
- When the change is ready: commit on the feature branch, push it right away, and open a ready-for-review PR (never a draft) with `gh pr create` unless the user explicitly asks not to.
- After opening a PR, monitor every required CI check to completion and report the outcome. **Merging is the maintainer's job — agents never merge PRs.** If a check fails, diagnose and fix it on the same branch, push the fix, and wait for the rerun to pass; a PR is ready to hand off only when every required check is green.
- After a PR merges, do not reuse its branch — start the next change from a fresh `main`.
- **External contributions are welcome and arrive as fork PRs.** A first-time contributor's checks sit in `action_required` until a maintainer approves the run (the button on the PR, or `gh api repos/windoze95/cantinarr/actions/runs/<id>/approve -X POST`), so a fork PR with no checks running is waiting on that, not broken. The Docker `build` job skips its GHCR `pr-N` tag push for forks (read-only token), so fork PRs have no preview image — the smoke tests still run. They hand off on the same all-green rule as any other PR; the maintainer merges.

## Verification

- Server changes: run `go vet ./...` and `go test ./...` from `server/`.
- App changes: run `flutter analyze --no-fatal-infos` and `flutter test` from `app/`. Golden tests live in `app/test` (committed `goldens/*.png` beside their test file); regenerate them with `flutter test --update-goldens` from `app/`.
- CI runs exactly those on every PR (Go tests with `-race`), plus a `CGO_ENABLED=0` server build and a `flutter build web --release`. A PR is not done if any of them fail. The same suite re-runs on every push to `main` and on a weekly schedule (toolchain drift); a red `main` run is a defect to fix promptly.
- **The `main` run is the release gate, not just an alarm.** It is the only run that ever tests the exact tree that ships: PR checks run against a merge preview computed from whatever `main` was at the time, and branch protection doesn't require branches to be up to date, so two independently green PRs can still land a broken `main`. Every shipping workflow — `docker.yml`, `testflight.yml`, `playstore.yml` — therefore opens with a `gate` job (`require-ci-green.yml`) that waits for the `CI` run on that exact SHA and fails the workflow if it isn't green. Nothing is published, uploaded, or tagged before that. A `v*` tag reuses the CI verdict from the `main` push of the same commit.
- Pull-request builds are deliberately *not* gated: a PR's Docker build is one of its own checks, and the `pr-N` preview image should stay pullable while the rest of the checks run. The gate only ever holds back an irreversible step.
- Codex integration changes are also proved against the checksum-verified pinned app-server in CI. The Docker workflow builds and smoke-tests both Dockerfiles, including bundled license notices, before publishing the root image to GHCR.
- iOS release builds happen only in CI (`testflight.yml`, auto-deploys on `main` when iOS-relevant `app/**` paths change — web/android/desktop subdirs, `app/test/**`, `app/tool/**`, markdown files, and store-listing metadata/screenshots are excluded; listing copy syncs via `storelisting.yml` instead). Don't assume a local iOS toolchain; when one isn't available, sanity-check Swift with `swiftc -parse` and let CI prove the build.
- iOS signing is manual, via the `IOS_PROVISIONING_PROFILE_BASE64` secret. Changing app capabilities/entitlements invalidates the profile — regenerate it and update the secret.
- Android release builds happen only in CI too (`playstore.yml`, builds a signed AAB on `main` when Android-relevant `app/**` paths change — web/ios/desktop subdirs, `app/test/**`, `app/tool/**`, markdown files, and store-listing metadata are excluded — and uploads it to the Play **alpha** track by default when `PLAY_SERVICE_ACCOUNT_JSON` is set; manual dispatch can pick `internal`). PRs that touch `app/android/**`, `app/pubspec.yaml`, or the workflow get a build-only check (no upload). No local Android SDK is assumed; let CI prove the build.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [windoze95/cantinarr](https://github.com/windoze95/cantinarr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
