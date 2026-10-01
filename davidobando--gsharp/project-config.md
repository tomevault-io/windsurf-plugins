---
trigger: always_on
description: Read [`CONTRIBUTING.md`](CONTRIBUTING.md) first. It covers building, testing,
---

# Working in this repository (for Claude Code and other agents)

Read [`CONTRIBUTING.md`](CONTRIBUTING.md) first. It covers building, testing,
discriminating tests (ADR-0154), design changes needing an ADR, and PR
hygiene. This file adds operational knowledge that has repeatedly cost time
when it was missing. When a rule here conflicts with a maintainer's explicit
instruction, follow the maintainer.

## Pull requests and review

- **Copilot reviews every push automatically.** Never request it, and never
  write `@copilot` anywhere (commits, PR bodies, comments). That mention
  summons a coding agent that pushes to the branch. Refer to it as "Copilot".
- **After every push, check the new review before calling the PR done.**
  - Confirm the review is on the current head. A review summary can belong to
    an older commit even when it appears after your push.
  - Fix what's real. Reply on each thread saying what changed and in which
    commit, then resolve it. If a finding is wrong, reply with the evidence
    rather than ignoring it.
  - Read the summary's "Open" and "Previously missed" sections too. They can
    hold real findings that have no inline thread.
- **`mergeStateStatus: CLEAN` ignores review threads.** Before merging, count
  unresolved threads across every page (replace `N`):
  ```sh
  gh api graphql --paginate -f query='query($endCursor: String) {
    repository(owner: "DavidObando", name: "gsharp") { pullRequest(number: N) {
      reviewThreads(first: 100, after: $endCursor) {
        nodes { isResolved } pageInfo { hasNextPage endCursor } } } } }' \
    --jq '.data.repository.pullRequest.reviewThreads.nodes[] | select(.isResolved == false) | 1' | wc -l
  ```
- **When review rounds keep finding the same class of bug in new shapes,
  stop patching shapes.** Find the single place the decision is made and fix
  it there, or add a fail-safe default. Long PRs have burned many rounds on
  shape-by-shape fixes.
- **Decide landing criteria when a PR runs long.** Block on anything that
  makes output wrong: an unsound or wrong assertion, double evaluation, or a
  runtime behavior change. File coverage gaps that fail loudly (a compile
  error a gate would catch) as follow-ups in the relevant tracking issue.
- **Commits and PR bodies carry no attribution or co-author lines.**
- **`gh pr edit --body-file` fails silently** (a stale Projects-classic
  GraphQL error). Use
  `gh api repos/DavidObando/gsharp/pulls/N -X PATCH -F body=@file`.
- **Release notes** live in `website/docs/release-notes.md` under
  Unreleased. Concurrent PRs all touch it:
  - after every rebase, re-read the whole file rather than just the conflict
    hunks;
  - resolve conflicts by keeping every bullet.
- **Out-of-scope findings become issues.** If a fix exposes an unrelated
  bug or a needed architectural change, file it (`gh issue create --label bug`)
  and link it from the PR instead of widening the PR or dropping it.

## CI gates and known behaviour

- **nullable-hygiene** runs `python3 build/nullable_hygiene.py --base
  origin/main`. Run it locally before pushing.
  - Every null-forgiving `!` needs an adjacent comment justifying it.
  - `= null!` initializers are rejected (ADR-0155).
  - Where a value really can be null, prefer restructuring over commenting
    the `!` away.
- **hot-core translation guard** (`build/run-cs2gs-selfmig-pr-guard.sh`)
  usually takes 30-60 minutes. It migrates a small set of net10 apps built
  against a fully annotated BCL. So it **cannot** catch regressions that only
  show up with unannotated or netstandard2.0 dependencies; issue #4361 is an
  example of one that slipped through.
- **Nightly self-migration** (`.github/workflows/cs2gs-selfmig-nightly.yml`)
  is the full-corpus gate, with a floor of all apps green. To verify a fix
  that affects migration, trigger it manually on `main` after merging:
  `gh workflow run cs2gs-selfmig-nightly.yml --ref main`.
- **cs2gs-oahu** migrates Oahu at the commit pinned in
  `tools/cs2gs/external/oahu.json`.
  `JobSchedulerTests.Bounded_Concurrency_Limit_Is_Enforced` is a known flaky
  test (DavidObando/Oahu#71). Show that a failure isn't yours before
  rerunning.
- **Build docs** (`pages.yml`): the WebKit dark-theme accessibility check on
  `project/quality-dashboard` fails intermittently. When a PR doesn't touch
  that page, rerun it with `gh run rerun <run-id> --failed`.
- **`Issue3347RemainingSpillInventoryTests`** translates code through cs2gs
  and fails if the output contains retired synthesized names. It has two
  scopes:
  - `src/Core` is checked for all four families: `__spill`, `__cast`,
    `__decon` and `__using`.
  - The cs2gs translator project is checked for `__spill` only.

  If it fails, change the new C# code (or cs2gs) so the translation doesn't
  produce the name. Don't weaken the test. Its translator scope doesn't cover
  the other three families, so don't treat it as a gate for them.
- **Flaky or not, check the logs first.** Pull the job log
  (`gh api repos/DavidObando/gsharp/actions/jobs/<id>/logs`) and show that
  the failure is unrelated, e.g. the same failure on `main` or on an
  unrelated PR, before rerunning.

## Local environment

- **Test scope:** run targeted `--filter` test runs locally. Full suites run

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DavidObando/gsharp](https://github.com/DavidObando/gsharp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
