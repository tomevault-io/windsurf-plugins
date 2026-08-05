---
trigger: always_on
description: This repository contains the Temporal Ruby SDK. Use this document as your quick reference when
---

# Contributor Guidance for `sdk-ruby`

This repository contains the Temporal Ruby SDK. Use this document as your quick reference when
submitting pull requests; use `README.md` and `CONTRIBUTING.md` for full contributor detail.

## Requirements for coding agents

- Run Ruby SDK commands from `temporalio/`.
- It is EXTREMELY IMPORTANT that any added comments should explain why something needs to be done,
  rather than what it is. Comments that simply state a fact easily understood from type signatures
  or other context should NEVER be added. Always prefer to avoid a comment unless it truly is 
  clarifying something nonobvious.
- `temporalio/ext/sdk-core/` has its own `AGENTS.md`; follow it understanding Core.
- Ignore generated/build output such as `temporalio/doc/`, `temporalio/pkg/`,
  `temporalio/target/`, `temporalio/tmp/`, `target/`, and `.bundle/`.
- Public API changes usually need tests, YARD/README updates, RBS, RBI, and a `CHANGELOG.md` entry
  when user-facing.
- Keep RBS in `temporalio/sig/` and RBI in `temporalio/rbi/` in sync with Ruby implementation
  changes.
- RBS is checked with `bundle exec rake steep`. RBI is validated by regular RBI parse/coverage tests
  and, when enabled, `TEMPORAL_SORBET_RUNTIME_CHECK=1 bundle exec rake test`, which applies RBI
  signatures at runtime through `test/support/sig_applicator.rb`.
- Do not increase manual RBI `T.untyped` or `T.anything` usage just to satisfy Sorbet. Use concrete
  types; only update the ratchet when usage decreases.
- When tests intentionally pass the wrong type, use `SigApplicator.suppress_errors` narrowly.
- Preserve compatibility-sensitive names and shapes: workflow names, activity names, payloads,
  errors, and option keywords.
- Activities and workflows receive payload arguments positionally and cannot accept keyword
  arguments.
- Add converter hints (`arg_hints`, `result_hint`, definition-level hints) when a new API path needs
  typed payload conversion.
- For workflow logic changes that could affect existing histories, add replay coverage with
  `Temporalio::Worker::WorkflowReplayer`.

## Repo Specific Utilities

- Target tests with `TESTOPTS`:

```bash
bundle exec rake test TESTOPTS="--name=test_some_method"
bundle exec rake test TESTOPTS="--name=/SomeClassName/"
TEMPORAL_SORBET_RUNTIME_CHECK=1 bundle exec rake test TESTOPTS="--name=test_some_method"
```

- Run `bundle exec rake rbs:install_collection` if Steep lacks its RBS collection.
- Tests start a local Temporal dev server by default. To use an existing server, set
  `TEMPORAL_TEST_CLIENT_TARGET_HOST` and optionally `TEMPORAL_TEST_CLIENT_TARGET_NAMESPACE`.
- Proto generation requires `protoc >= 34.0` and `protoc-gen-rbi`. From `temporalio/`, install the
  RBI plugin with `go install github.com/coinbase/protoc-gen-rbi@$(cat ../.protoc-gen-rbi-version)`,
  or set `PROTOC_GEN_RBI` to an executable path.

## Building and Testing

Run the following commands from `temporalio/`:

```bash
bundle exec rake test                         # run the Minitest suite
TEMPORAL_SORBET_RUNTIME_CHECK=1 bundle exec rake test  # run tests with RBI runtime validation
bundle exec rake rubocop                      # run Ruby linting
bundle exec rake steep                        # run RBS checks
bundle exec rake proto:generate               # regenerate protobuf/RBI/RBS output
bundle exec rake proto:check_generated        # verify generated output is current
bundle exec rake compile                      # compile the native bridge
bundle exec rake rust_lint                    # lint Rust bridge changes
bundle exec rake yard                         # generate/check public API docs
bundle exec rake                              # run the default local verification suite
```

The default `bundle exec rake` runs RuboCop, YARD, RBS collection install, Steep, Rust bridge lint,
compile, and the full Minitest suite. It does not enable Sorbet runtime validation.

## Expectations for Pull Requests

- Format and lint your code before submitting.
- Ensure targeted tests cover the changed behavior.
- Update RBS/RBI signatures with Ruby implementation changes.
- Update docs and `CHANGELOG.md` for public user-facing API changes.
- Regenerate protobuf output with `bundle exec rake proto:generate`

## Review Checklist

Reviewers will look for:

- Targeted tests covering behavior changes.
- `bundle exec rake rubocop` passing.
- `bundle exec rake steep` passing for RBS/API changes.
- `TEMPORAL_SORBET_RUNTIME_CHECK=1 bundle exec rake test` passing for RBI/API changes.
- `bundle exec rake proto:check_generated` passing for proto/generated changes.
- `bundle exec rake compile` and `bundle exec rake rust_lint` passing for bridge changes.
- `bundle exec rake yard` passing and docs/changelog updated for public user-facing changes.

## Where Things Are

- `temporalio/lib/temporalio/` - Ruby SDK implementation.
- `temporalio/sig/` - RBS signatures.
- `temporalio/rbi/` - Sorbet RBI signatures.
- `temporalio/test/` - Minitest suite and helpers.
- `temporalio/ext/src/` - Rust extension bridge.
- `temporalio/extra/` - generators and support scripts.
- `temporalio/ext/sdk-core/` - embedded Temporal Core SDK with its own `AGENTS.md`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [temporalio/sdk-ruby](https://github.com/temporalio/sdk-ruby) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
