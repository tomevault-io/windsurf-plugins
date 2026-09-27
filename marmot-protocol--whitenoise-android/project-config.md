---
trigger: always_on
description: The Android app must not create its own persistent data cache for White Noise protocol data.
---

# White Noise Android Agent Notes

## Source Of Truth

The Android app must not create its own persistent data cache for White Noise protocol data.

White Noise's SQLite databases are the source of truth for accounts, groups, messages, profiles, relay state, key packages, notification settings, and other protocol data. Performance work should make reads from that source fast and well-shaped. Do not add a second Android-owned cache to hide slow database or binding access.

Allowed Android state:

- UI state, such as selected screens, dialog state, draft text, loading flags, and optimistic send state.
- Short-lived lifecycle state, such as active stream jobs and in-flight requests.
- Android platform preferences, such as theme, language, notification permission UI state, and foreground service preferences.

Avoid:

- Room, DataStore, SharedPreferences, files, or in-memory singleton maps as a duplicate source for White Noise protocol data.
- Long-lived Android caches for message timelines, chat summaries, profiles, key packages, or relay projections.
- Fixing slow screens by copying White Noise protocol data into Android-owned storage.

Prefer:

- Changing White Noise APIs so Android can read exactly the data it needs from SQLite quickly.
- Moving binding and database calls off the main thread.
- Keeping stream subscriptions lifecycle-bound and explicitly closed.
- Returning pre-shaped projections from White Noise when a screen needs them.

If Android seems to need a cache for protocol data, stop and ask whether the data should instead be indexed, projected, or exposed differently by White Noise.

## Working Rules

- Keep Android changes small and tied to a user-visible behavior.
- Use the existing Kotlin, Compose, and Marmot binding patterns before adding new architecture.
- Do not run connected device tests that can wipe local app state unless the user asks for them.
- Prefer `rg` for searching.
- Before editing files, check the dirty worktree and preserve changes you did not make.

## Release operations

Read [`docs/android-release-pipeline.md`](docs/android-release-pipeline.md) before
preparing or distributing a release. It is the canonical operator runbook for
versioning, signed builds, candidate verification, Play internal uploads, GitHub
drafts, and the separate public Zapstore gate. Distribute the reviewed candidate
without rebuilding it. A build or internal-testing request does not authorize
public Zapstore publication. Record the source commit, build run/attempt,
manifest digest, artifact hashes, and destination readback for each release.

## Manual release testing guide

[`docs/manual-release-testing.md`](docs/manual-release-testing.md) is the canonical novice-facing release checklist. Every change to a user-visible route, label, action, state, permission, notification, Android entry point, supported configuration, or failure/recovery path must update the corresponding permanent test IDs in the same pull request. Add a new ID for new behavior; never renumber or reuse an ID. When behavior is removed, move its ID to the Retired IDs table and record the pull request or commit that removed it.

Keep [`docs/manual-release-testing-surfaces.json`](docs/manual-release-testing-surfaces.json) synchronized with source symbols and active test IDs. Run `python3 scripts/check_manual_test_guide.py` and `python3 -m unittest scripts/test_check_manual_test_guide.py` after modifying the guide, its inventory, or a user-facing surface. The guide's boxes stay unchecked in Git; testers copy the checklist elsewhere for a run.

## Visual Change Evidence

- Every pull request that changes rendered UI must add or update a deterministic
  Roborazzi screenshot test and commit the generated PNG baseline under
  `app/src/test/snapshots/`.
- Cover the states materially affected by the change. Include relevant themes,
  RTL, large font scale, loading/empty/error states, and loaded/fallback content
  when those states can alter the result; do not create unrelated snapshot
  variants merely to increase coverage.
- After every commit that changes rendering, regenerate and commit the affected
  baselines with `./gradlew :app:recordRoborazziDevZapstoreDebug` or
  `:app:recordRoborazziDevPlayDebug`. Before pushing, run
  `./gradlew :app:verifyRoborazziDevZapstoreDebug
  :app:verifyRoborazziDevPlayDebug`.
- Verify that the pull request description's generated **Visual changes** section
  points to the exact current head commit and shows every affected baseline.
- Treat a missing-screenshot CI result as blocking. Do not dismiss it as an
  informational warning or bypass it by changing unrelated snapshots.

## GitHub triage

- The public White Noise Android project is
  `https://github.com/orgs/marmot-protocol/projects/7`.
- Use the authenticated `gh` CLI for repository and GitHub Project operations.
  A GitHub MCP connector or plugin is not required. Start with `gh auth status`
  and require `repo`, `project`, `read:org`, and `workflow` scopes.
- A sandbox or isolated worktree must preserve access to the existing gh config
  and network. Do not conclude that GitHub is unavailable merely because an MCP
  tool is absent.
- Read `.agents/issue-triage.md` before creating or materially editing an issue.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [marmot-protocol/whitenoise-android](https://github.com/marmot-protocol/whitenoise-android) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
