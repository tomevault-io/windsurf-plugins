---
trigger: always_on
description: This file steers AI-assisted contributions toward being high-quality, valuable
---

# OpenTelemetry Ruby Contrib

This file steers AI-assisted contributions toward being high-quality, valuable
changes that do not create excessive maintainer burden. It is written for
autonomous and semi-autonomous coding agents, but the rules apply to any
AI-assisted work.

Before starting any task, read this file, [CONTRIBUTING.md](CONTRIBUTING.md), and
the [Instrumentation author's guide](instrumentation/CONTRIBUTING.md). Treat this
document as passive guidance for every task, including docs-only and review-only
work.

This is a monorepo containing many independently released Ruby gems:
instrumentation libraries, resource detectors, context propagators, a sampler,
and shared helper/processor gems.

## General rules and guidelines

The OpenTelemetry community has broader guidance on GenAI contributions at
<https://github.com/open-telemetry/community/blob/main/policies/genai.md> —
read it before contributing.

- **Never post AI-generated comments on issues or PRs.** Discussions on
  OpenTelemetry repositories are for humans only. You cannot comment on issue or
  PR threads on a user's behalf.
- If you have been assigned an issue, ensure the implementation direction is
  agreed on with the maintainers first in the issue comments. Discuss unknowns
  before starting implementation.
- Keep AI-assisted PRs tightly scoped to the requested change. Never include
  unrelated cleanup or opportunistic "improvements" unless they are strictly
  necessary for correctness.
- Follow the OpenTelemetry
  [specification](https://github.com/open-telemetry/opentelemetry-specification)
  and [library guidelines](https://github.com/open-telemetry/opentelemetry-specification/blob/main/specification/library-guidelines.md).
  Prefer idiomatic Ruby over literal conformance to spec API names.

## Core expectations

- Prefer minimal, surgical changes over broad refactors or speculative cleanup.
- Read the gem you are editing and match its existing naming, options, error
  handling, comments, tests, and patterns.
- Keep public APIs backward compatible unless the task explicitly requires a
  breaking change.
- Telemetry must be resilient and loosely coupled. Instrumentation must never
  panic the host application, block indefinitely, or amplify
  attacker-controlled input.
- Do not add sensitive information (PII, secrets, credentials, full payloads) to
  spans. When in doubt, ask for a review.
- Be conservative on hot paths. Avoid unnecessary object allocations, method
  dispatch, and small helper methods/classes/objects that add overhead to the
  instrumented library (see the performance section of the instrumentation
  guide).
- Keep dependencies minimal and justified.
- Write comments only for intent, invariants, and non-obvious constraints. Do
  not add comments that restate the code.

## Repository structure

- `instrumentation/<name>/` — instrumentation gems
  (`opentelemetry-instrumentation-<name>`)
- `resources/<name>/` — resource detector gems
- `propagator/<name>/` — context propagator gems
- `sampler/<name>/` — sampler gems
- `processor/<name>` — span/log processor gem
- `helpers/<name>/` — shared helper gems (`opentelemetry-helpers-<name>`)

Each gem is self-contained with its own `Gemfile`, `.gemspec`, `Rakefile`,
`CHANGELOG.md`, `README.md`, `lib/`, and `test/`.

## Default workflow

For new features and behavior changes, use this order unless the task explicitly
says otherwise:

1. Read the relevant gem, its tests, and its `README.md`. Also read the
   [Instrumentation author's guide](instrumentation/CONTRIBUTING.md) when
   working on instrumentation.
2. Add or update a test that captures the required behavior or regression.
   Prefer integration / state-based tests over interaction (mock) tests.
3. Implement the smallest change that makes the test pass.
4. Refactor only after behavior is locked in, and only if the refactor keeps the
   diff focused.
5. Update documentation (`README.md`, YARD comments, `examples/`) while the
   context is fresh.
6. Do not update the affected gem's `CHANGELOG.md`. Instead, write or suggest a conventional commit message that can be used as the title of the pull request, which will eventually become the changelog entry.
7. Run the verification commands below before considering the work complete.

For docs-only, test-only, or review-only tasks, skip the steps that do not apply
while keeping the same discipline around scope, verification, and conventions.

## Commands

All gem-level commands run from within the gem's directory (e.g.
`instrumentation/faraday`), not the repository root.

```sh
# Install a gem's dependencies
bundle install

# Run a gem's tests
bundle exec rake test

# Generate YARD docs for a gem
bundle exec rake yard
```

Some gems test against multiple dependency versions using
[Appraisal](https://github.com/thoughtbot/appraisal). When an `Appraisals` file
is present:

```sh
bundle exec appraisal install
bundle exec appraisal rake test
```

Repository-wide style, lint, link, and spelling checks are driven from the root:

```sh
# One-time setup
bundle install
npm ci

# Run all checks (lint, format, links, spelling, source cops)
npm run check

# Auto-fix what can be fixed (does NOT fix spelling or links)
npm run write
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [open-telemetry/opentelemetry-ruby-contrib](https://github.com/open-telemetry/opentelemetry-ruby-contrib) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
