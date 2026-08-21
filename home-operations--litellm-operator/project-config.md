---
trigger: always_on
description: Guidance for AI coding agents (and humans) writing Go in this repository. This
---

# AGENTS.md: litellm-operator

Guidance for AI coding agents (and humans) writing Go in this repository. This
file is meant to be identical, or nearly so, across every Go service/CLI in
the home-operations fleet: treat it as a template, copy it verbatim into a
new Go repo, and only add repo-specific detail if something here genuinely
doesn't apply. Where a fact could differ per repo (exact versions, task
commands, variable names, CI steps), this file points at where to check
rather than asserting a specific value, since restating it here just goes
stale.

## Working in this repo: AI usage, commits, and safety

This repo doesn't carry its own `CONTRIBUTING.md`; GitHub serves the org-wide
one from [`home-operations/.github`](https://github.com/home-operations/.github/blob/main/CONTRIBUTING.md),
which includes an AI Usage Policy that applies to any AI coding agent here:
assistive use only, a human must author the majority of any change, AI use
must be disclosed, a human reviews every line before submission, and the
contributor must be able to explain any line a reviewer asks about. AI must
never write the PR description, an issue, or a reply to a human on the
contributor's behalf. Read the policy itself rather than trusting this
summary; it can change without this file being updated to match.

- PR titles follow [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/):
  `<type>[(scope)][!]: <description>` (e.g. `fix(config): reject a negative
timeout`), which is what drives release-please's version bumps. Individual
  commit messages don't have to follow the format, though matching it is
  fine. Sign off commits: `git commit -s`.
- Never `git commit`, `git push`, or open a PR unless asked to. Ask before
  any destructive or hard-to-reverse action (force-push, `git reset --hard`,
  deleting a branch, rewriting history) instead of defaulting to it.
- Never touch secrets or gitignored files. Check this repo's actual
  `.gitignore` before assuming it already excludes something like
  `*.key`/`*.crt`/`.env`; don't assume gitignore coverage that isn't
  actually configured. This fleet generally passes signing keys and webhook
  secrets by path or env var specifically so they're never committed; don't
  be the exception.
- Don't state a library's API, flags, or defaults from memory: verify
  against `pkg.go.dev`, the vendored source in the module cache, or this
  project's own code. Dependency behavior changes between versions in ways
  that are easy to get subtly wrong from recollection alone.
- After a change, run this repo's actual test and lint tasks (see "Build,
  lint, test" below) before calling it done. Don't claim untested code
  works.

## Baseline

- **Idiomatic.** Follow [Effective Go](https://go.dev/doc/effective_go) and
  the [Code Review Comments](https://go.dev/wiki/CodeReviewComments) wiki.
  `gofmt -s` runs on every staged `.go` file via lefthook and again in CI:
  never hand-format, and don't fight it with inline exceptions. Comments
  explain non-obvious constraints only (a hidden invariant, why a workaround
  exists, what would surprise a reader); don't narrate what good naming
  already says, and don't reference the current change or past behavior in
  a comment: that belongs in the PR description and rots as the code moves
  on.
- **Go 1.26.** `go.mod`'s `go` directive is pinned to the lowest 1.26
  release the dependencies allow (1.26.0 unless one forces higher) and
  Renovate no longer bumps it; `.mise/config.toml`'s `tools.go` is the
  toolchain that actually builds and tests, so the two are expected to
  differ. Raise the directive only when the code or a dependency needs a
  newer Go version. When a newer construct is
  genuinely more idiomatic, use it: Go 1.26 added `errors.AsType[T](err)`, a
  generic type-safe replacement for the `var t *T; errors.As(err, &t)`
  two-step; prefer it in new code. `go fix` (rebuilt in 1.26 as a
  modernizer runner on `go vet`'s analysis) surfaces these mechanical
  migrations; run it after a toolchain bump.
- **Idempotent.** Reconcilers, code generators (`mise run generate`), and
  CLI subcommands must be safe to re-run: identical input yields identical
  output/state, with no accumulating side effects on a second invocation.
  The strongest version of this is a stateless service: if every response
  is re-derivable from its inputs or upstream, a restart or an extra
  replica can't affect correctness, only latency.
- **DRY and minimal, without premature abstraction.** Three similar call
  sites are fine as-is; don't introduce an interface, options struct, or
  generic helper until a real third caller needs the variance it buys.
  Touch only what the task requires: don't refactor or "improve" adjacent
  code, and match the existing style even where you'd do it differently.
  Remove imports, variables, and functions your own change orphaned; leave
  pre-existing dead code alone and mention it instead of deleting it
  unprompted.
- **Unit tested**, table-driven via `t.Run` subtests. Match whatever
  framework the package you're touching already uses instead of assuming:
  plain stdlib `testing` is the fleet default and is sufficient for most
  tests (config parsing, HTTP handlers, pure functions); `testify`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [home-operations/litellm-operator](https://github.com/home-operations/litellm-operator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
