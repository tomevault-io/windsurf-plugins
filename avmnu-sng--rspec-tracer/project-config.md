---
trigger: always_on
description: Guidance for Claude Code (and other AI coding agents) working in this
---

# CLAUDE.md

Guidance for Claude Code (and other AI coding agents) working in this
repository.

## `docs/revamp/` is local-only — never commit it

The `docs/revamp/` tree is the maintainer's internal 2.0 planning material
(milestone briefs, session plans, architecture notes). It is listed in
[`.gitignore`](.gitignore) and must stay there.

- **Do not** `git add docs/revamp/` or any file under it, even if asked
  indirectly ("commit everything", "stage all changes", etc.). Always
  exclude the path explicitly.
- **Do not** copy, paraphrase, or quote content from `docs/revamp/` into
  files that *are* committed (README, CHANGELOG, CLAUDE.md, commit
  messages, PR descriptions).
- When the maintainer directs you to one of those files (e.g. "read
  `docs/revamp/sessions/M1.1-bug-analysis-fixes.md`"), use it as
  conversation context only. The resulting PR must stand on its own
  without any `docs/revamp/` reference.

If the file is missing on a fresh clone, don't re-create it — ask the
maintainer for a local copy.

## Project

`rspec-tracer` — a Ruby gem that speeds up RSpec runs by tracking which
source files each test depends on, so subsequent runs only re-run affected
tests.

A 2.0 revamp is in progress. The authoritative plan lives in the local
`docs/revamp/` tree (see the banner above). This file, the code, and
commit messages are the only materials that survive in the repository.

## Rules of engagement (from the revamp plan)

- **No blind cherry-picks.** Evidence of bugs (issues, fork PRs) is
  *input*; fixes are designed against the current architecture, not
  copied.
- **One session = one PR-sized deliverable.** Don't scope-creep.
- **Every acceptance criterion is mechanically checkable.** If you can't
  check it, rewrite it.
- **Graceful degradation always.** Never propagate a tracer failure into
  the user's test suite.
- **Maximum supported surface.** Drop something only for a genuine
  technical limitation (Windows path semantics, ENV-driven branches with
  no declaration, refinements in unexecuted files). "Effort" is not a
  reason.
- **The feedback loop is sacred.** `task check` (lint + unit specs +
  smoke benchmark) is the dev-loop command; it must stay under ~10 s.
  Any regression is the highest-priority fix.

## Dev loop

- `task check` — the fast feedback loop: `task lint:ruby` + `task test:unit`
  + `task benchmark:smoke`. Runs in ~2 s post-M2.4.
- `task ci` — full CI pipeline locally (lint + check + test + security +
  benchmark + integration).
- `task --list` — full catalog. Common: `task lint:all`, `task test:unit`,
  `task test:features:{ruby,ruby-parallel,rails,jruby}`,
  `task benchmark:{smoke,full,ratchet:update}`,
  `task fixtures:rails:{bundle,migrate,seed,rspec,lint}`,
  `task security:{bundle-audit,trivy}`,
  `task coverage:{merge,clean}`.
- **Coverage**: SimpleCov config lives inline in `spec/spec_helper.rb`
  (NOT in a `.simplecov` file at repo root — SimpleCov's `.simplecov`
  auto-loader walks upward from `Dir.pwd` and would leak our config
  into fixture subprocesses that `chdir` into `spec/fixtures/rails_app/`
  or `benchmark/fixtures/ruby_app/`, causing 3x benchmark regressions).
  Per-suite command_names come from the `COVERAGE_SUITE` env (each
  `task test:*` sets it). Multi-suite resultsets auto-merge into
  `coverage/.resultset.json`; `task coverage:merge` collates them via
  `SimpleCov.collate` and produces `coverage/index.html` +
  `coverage/coverage.json`. CI: `lint-and-specs.yml`'s `coverage` job
  downloads per-job artifacts + pushes the merged JSON to Codecov
  (sole public consumer; the gh-pages `/coverage/` page used to mirror
  this and was removed since Codecov is the canonical surface). Skip
  with `COVERAGE=false` env; mutation runs auto-skip via
  `defined?(::Mutant)`.
- `bundle exec rake` is still wired (legacy cucumber path). Removed in
  the 2.0.0 cleanup.

Tool versions are pinned at the top of `Taskfile.yml` (`vars:`); bump
them in one place.

### Prefer `task <name>` over raw `bundle exec …`

The Taskfile is the canonical entry point; each task bakes in the
right env (`RSPEC_TRACER_DISABLE`, paths, flags) and guards (e.g.
mutant Ruby-version check). Reaching past it to `bundle exec rspec …`
bypasses those guards and masks issues that CI will catch. Use direct
`bundle exec` only for targeted debugging (one spec file, one flag) —
always re-run the relevant `task` before committing.

### Pre-PR local parity

`task check` is necessary but **not sufficient** — it only runs
`lint:ruby + test:unit + benchmark:smoke`. Before opening a PR, also
run every task the per-PR CI (`.github/workflows/lint-and-specs.yml`)
runs:

```
task lint:ruby          # already in task check
task lint:actions       # actionlint
task lint:yaml          # yamllint --strict
task check:bundle       # Gemfile.lock installable
task security:bundle-audit
task security:trivy
task test:unit          # already in task check
task test:property      # rantly properties
task test:mutation:smoke # ≥ 90% on TimeFormatter.format_time
task test:dogfood       # tracer-on-tracer subprocess smoke
task benchmark:smoke    # already in task check
```

Why: `task check`'s narrow surface is intentional (10 s budget). The

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [avmnu-sng/rspec-tracer](https://github.com/avmnu-sng/rspec-tracer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
