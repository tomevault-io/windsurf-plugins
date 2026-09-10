---
trigger: always_on
description: Context for contributors on how this SDK is put together, plus the things reviews most often come back to. Setup instructions live in [CONTRIBUTING.md](CONTRIBUTING.md).
---

# Working in this repository

Context for contributors on how this SDK is put together, plus the things reviews most often come back to. Setup instructions live in [CONTRIBUTING.md](CONTRIBUTING.md).

## Overview

- The official Ruby SDK for the Claude API (the `anthropic` gem). Much of it is produced via code generation from the OpenAPI spec, with hand-written helpers (`lib/anthropic/helpers/**`: streaming, tools, input schemas, platform clients), credentials, middleware and examples layered on top.
- Generated files are safe to edit. New generator output is git-merged with custom changes rather than overwriting them, so fix things where they live; a collision is just a merge conflict. (`git diff origin/generated origin/next` shows everything custom if you ever need to know which is which.)
- Fix things upstream first wherever you can, before reaching for custom code: a missing parameter, an out-of-date doc string, or a response the models can't represent (a missing required field, an unknown enum member, a nullability mismatch) belongs in the OpenAPI spec, and broken generated infrastructure (`lib/anthropic/internal/**`: transport, retries, SSE and multipart handling, coercion) belongs in the generator — fixing it there is preferred over a local patch.
- PRs target `next`. `main` only moves when a release is cut. `next` moves quickly and its history is occasionally rewritten, so rebase onto the current `origin/next` before asking for review — a stale base shows up as unrelated "changed" files (`git rebase --onto origin/next <old-base>` sorts out a base that was rewritten underneath you).
- `lib/anthropic/version.rb`, `.release-please-manifest.json`, the gem's own entry in `Gemfile.lock`, the `README.md` version block and `CHANGELOG.md` are written by release automation.

## Build, test, lint

- `./scripts/bootstrap` installs the bundle; `./scripts/test` starts the mock API server (`./scripts/mock`, needs Node) and runs the suite; run `./scripts/lint` before pushing (it is exactly what CI runs) and format only the files you touched, e.g. `bundle exec rubocop -a <files>` — a whole-tree `./scripts/format` also reflows long lines and restyles `rbi/`/`sig/` files you never opened, since CI doesn't check formatting. A single file runs with `bundle exec rake test TEST=test/anthropic/…_test.rb`.
- Lint is rubocop plus `steep check` (RBS) plus Sorbet, and the Sorbet step is `srb typecheck --dir examples` — the examples are type-checked in CI on top of `rbi/`. A bare `bundle exec srb tc` never looks at `examples/`, which is how "green locally, red in CI" has happened; use the `--dir examples` form after touching examples or any signature they use. Rubocop runs with `Layout/LineLength` excluded, so line length is never what fails CI; trailing whitespace (easy to pick up while resolving a sync conflict) and missing parentheses are.
- Only the generated `ResourceTest` suites under `test/anthropic/resources/**` need the mock server (port 4010, or `TEST_API_BASE_URL`); `APIConnectionError`s there mean the mock isn't running. Everything else runs offline against WebMock — helper, credentials, middleware and client tests, plus the hand-written `resources/messages/streaming_test.rb` and `resources/beta/messages/streaming_test.rb` that sit inside that directory. Live tests against real cloud providers only run with `ANTHROPIC_LIVE=1`, and the Bedrock/AWS unit tests read ambient AWS state (`AWS_*` variables and `~/.aws` profiles), so if they fail locally on region, profile or credential lookup, run them with `AWS_*` unset and `AWS_CONFIG_FILE=/dev/null AWS_SHARED_CREDENTIALS_FILE=/dev/null`.
- CI's `./scripts/detect-breaking-changes` restores the release base's generated tests and `client_test.rb` and re-runs `./scripts/lint` over them, but nothing in lint type-checks `test/`, so it will not notice a removed or renamed method or keyword — treat public-surface removals as a review item: deprecate first (a forwarding alias plus `warn(…, category: :deprecated)`) and remove later, in step with the other SDKs.
- The gem supports Ruby 3.2+, and users also run it on Ruby 4 (Prism) and JRuby, which CI doesn't cover; both have broken on load-time edge cases before (bytecode precompilation of unusual syntax, constant resolution during `require`), so keep load-time code boring and take reports from those runtimes seriously.
- Examples are executable scripts (`#!/usr/bin/env ruby`, `# frozen_string_literal: true`, a `# typed:` sigil, `require_relative "../lib/anthropic"`), linted and type-checked with everything else. Match the sigil of neighbouring examples of the same kind: client and streaming examples are `typed: strong`, while the `BaseModel`/`BaseTool` DSL examples (tools, tool runner, structured output, input schemas) can't type-check today and stay `typed: false` or unsigilled, where Sorbet checks little beyond syntax and constant names. New user-facing helpers usually come with an example and a `helpers.md` section, using a current public model id like the neighbouring examples; user documentation beyond `helpers.md` lives outside this repo (see CONTRIBUTING.md), so say in the PR what needs documenting.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [anthropics/anthropic-sdk-ruby](https://github.com/anthropics/anthropic-sdk-ruby) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
