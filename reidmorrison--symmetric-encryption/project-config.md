---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

`symmetric-encryption` is a Ruby gem (currently v5.0.0, in development on branch `feature/housekeeping`) that encrypts ActiveRecord attributes, Mongoid fields, passwords in config files, and whole files/streams using OpenSSL. Minimum Ruby 3.2, minimum Rails/ActiveRecord 7.2.

Public docs live in [docs/](docs/) and are published to https://encryption.reidmorrison.com/ via GitHub Pages (Jekyll front-matter in each file). Update these when changing user-facing behavior.

## Commands

```bash
docker compose up -d                       # Start MongoDB, needed by test/mongoid_test.rb
bundle exec rake test                      # Run the suite against the default Gemfile
bundle exec rake test TEST=test/cipher_test.rb
bundle exec ruby test/cipher_test.rb -n "/permit replacing value/"   # Single test by name

bundle exec rake                           # Rubocop, then the suite for every appraisal. Use this to verify a change.
appraisal install                          # Regenerate gemfiles/*.gemfile and install
appraisal rails_8.1 rake test              # One Rails version (rails_7.2, rails_8.0, rails_8.1)

COVERAGE=true bundle exec rake test   # Writes coverage/index.html (currently ~96% line coverage)

bundle exec rubocop                        # Also runs first as part of the default rake task
bundle exec rubocop -a                     # Safe autocorrections only. Check -A suggestions by hand.
bundle exec solargraph typecheck            # Optional, MRI only
```

SimpleCov is off unless `COVERAGE` is set, and is started at the top of `test_helper.rb` before any lib file is required so that untouched files still count.

`rake test` runs one version for a quick check; bare `rake` runs Rubocop and then fans out to every appraisal, because the default task delegates to `appraisal` unless `APPRAISAL_INITIALIZED` or `TRAVIS` is set. Rubocop is not part of the inner default task, so it runs once rather than once per appraisal, and offenses abort the run before any test starts. Run `bundle exec rake` before calling a change done.

Tests are Minitest with the spec DSL (`describe`/`it`). [test/test_helper.rb](test/test_helper.rb) loads [test/config/symmetric-encryption.yml](test/config/symmetric-encryption.yml) with env `test` and chmods the test key files to 0600 (git does not preserve the mode, and `Keystore::File#read` refuses to read a key file with looser permissions).

Two things to know before writing tests here:

- `SymmetricEncryption.cipher` is module-level global state. Anything that calls `Config.load!` (directly, or through the CLI) replaces the ciphers `test_helper` set up and breaks whichever file runs next, since Minitest randomizes order. Save and restore `cipher` and `secondary_ciphers` around such tests, as [cli_test.rb](test/cli_test.rb) does.
- Minitest rejects `let` names that begin with `test` or that shadow a `Minitest::Spec` method (`value`, `name`, ...). That is why the existing helpers are named `the_test_path`, `the_config_file_name`, and so on.

Two groups of tests skip themselves rather than fail, so watch the run count:

- The credentialed cloud keystore tests ([keystore/aws_test.rb](test/keystore/aws_test.rb), [keystore/gcp_test.rb](test/keystore/gcp_test.rb), [utils/aws_test.rb](test/utils/aws_test.rb)) skip unless AWS or GCP credentials are set. The logic they cover is also tested offline, see below.
- [test/mongoid_test.rb](test/mongoid_test.rb) pings MongoDB at load time and skips the whole file with an explanatory message when the gem is missing or the server is unreachable. Start MongoDB with `docker compose up -d` ([docker-compose.yml](docker-compose.yml)) before expecting those 58 tests to run.

The cloud keystores are covered offline by the `*_stubbed_test.rb` files, which need no credentials and make no network calls:

- AWS uses the SDK's own response stubbing (`Aws.config[:stub_responses]`). Prefer this over hand-written mocks: request parameters are still validated against the real KMS API model, so a misnamed argument fails the test.
- Cloud KMS has no equivalent, so [keystore/gcp_stubbed_test.rb](test/keystore/gcp_stubbed_test.rb) replaces the client with a stub that returns the real response protobufs and records the request arguments.

CI ([.github/workflows/ci.yml](.github/workflows/ci.yml)) runs Rails 7.2/Ruby 3.2, Rails 8.0/Ruby 3.4, Rails 8.1/Ruby 4.0 with a mongo service, via `BUNDLE_GEMFILE=gemfiles/rails_X.Y.gemfile bundle exec rake test`. Rubocop is a separate single job rather than a fourth matrix entry, since its result does not depend on the Rails version.

## Architecture

### Entry points

[lib/symmetric_encryption.rb](lib/symmetric_encryption.rb) (and its alias [lib/symmetric-encryption.rb](lib/symmetric-encryption.rb)) is the full entry point: it loads core, then optionally hooks Rails via the Railtie and ActiveRecord/Mongoid via `ActiveSupport.on_load`, so gem load order does not matter. [lib/symmetric_encryption/core.rb](lib/symmetric_encryption/core.rb) is the framework-free entry point for standalone Ruby apps and declares nearly everything else as `autoload`; add new files there.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [reidmorrison/symmetric-encryption](https://github.com/reidmorrison/symmetric-encryption) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
