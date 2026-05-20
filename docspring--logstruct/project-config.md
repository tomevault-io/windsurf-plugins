---
trigger: always_on
description: 1. **NEVER mark a feature as done until `task ci` is passing**
---

# LogStruct Development Guide

## 🚨 CRITICAL RULES - MUST ALWAYS BE FOLLOWED 🚨

1. **NEVER mark a feature as done until `task ci` is passing**
2. **ALWAYS run `task ci` before claiming completion**
3. **NO EXCEPTIONS to the above rules - features are NOT complete until all checks pass**
4. **This rule must ALWAYS be followed no matter what**
5. **`additional_data` is ONLY for unknown and dynamic fields that a user or integration might send. IF YOU KNOW THE NAME OF A FIELD, IT GOES IN THE SCHEMA. NEVER use `additional_data` for known fields.**
6. **NEVER run `git tag` without `-m` flag** - it opens an interactive editor and freezes. Always use: `git tag -a v0.1.7 -m "Release v0.1.7"`

## Commands

All commands use [Task](https://taskfile.dev) - run `task --list` to see all available tasks.

### Core Commands

- Setup: `scripts/setup.sh`
- Run full CI validation: `task ci` (typecheck, lint, spellcheck, tests, etc.)
- Auto-fix lint issues then run CI: `task fix`
- Interactive console: `scripts/console.rb`

### Testing Commands

- Run all tests (unit + Rails integration): `task test`
- Run Ruby unit tests only: `task ruby:test`
- Run Rails integration tests: `task rails:test`
- Run Rails tests for all supported versions: `task rails:test:matrix`
- Run frontend tests: `task docs:test`
- Run single test file: `bundle exec ruby scripts/test.rb test/path_to_test.rb`
- Run test at specific line: `bundle exec ruby scripts/test.rb test/path_to_test.rb:LINE_NUMBER`
- Run test by name: `bundle exec ruby scripts/test.rb -n=test_method_name`
- Debug a specific test: Add `debugger` statements (developer only)
- Merge coverage reports: `task ruby:coverage:merge`

**Rails Integration Tests**: Located in `rails_test_app/templates/test/integration/`. The script automatically copies template files to the test app when you run `scripts/rails_tests.sh`.

**FORCE_RECREATE**: Do NOT use `FORCE_RECREATE=true` unless absolutely necessary. It deletes the entire test app and recreates from scratch, which is slow. Only needed when:
- Gemfile changes require reinstalling gems
- Something is fundamentally broken with the cached app
- Rails version changes (but the script auto-detects this anyway)

For template file changes (tests, rake tasks, etc.), just run `scripts/rails_tests.sh` - it copies templates automatically.

### Quality Commands

- Run all typechecking: `task typecheck`
- Run all linting: `task lint`
- Auto-fix all lint issues: `task lint:fix`
- Spellcheck: `task spellcheck`

#### Ruby Commands

- Ruby typecheck: `task ruby:typecheck`
- Lint Ruby: `task ruby:lint`
- Format Ruby: `task ruby:lint:fix`

#### Frontend Commands

- Next.js typecheck: `task docs:typecheck`
- Lint frontend: `task docs:lint`
- Auto-fix frontend lint: `task docs:lint:fix`

#### Formatting Commands

- Format all files (JS/TS/JSON/YAML/MD): `task prettier:write`
- Check formatting: `task prettier:check`

### Development Commands

- Generate all (structs + catalog): `task generate`
- Generate Sorbet RBI files: `task ruby:tapioca`
- Generate spellcheck dictionary: `scripts/generate_lockfile_words.sh`
- Check generated files are up to date: `task ruby:check-generated`

### Documentation Commands

- Generate YARD docs: `task yard:generate`
- Clean YARD docs: `task yard:clean`
- Regenerate YARD docs: `task yard:regen`
- Open YARD docs in browser: `task yard:open`

### Gem Commands

- Build gem: `task gem:build`
- Install gem locally: `task gem:install`
- Install gem without network: `task gem:install:local`
- Release gem to RubyGems: `task gem:release`

## Terraform Provider repo in this workspace

- The Terraform provider lives in a separate GitHub repo: `DocSpring/terraform-provider-logstruct`.
- For convenience, the provider repo is checked out as a plain directory at `./terraform-provider-logstruct/` in this repo. It is NOT a Git submodule and is ignored by this repo’s `.gitignore`.
- You can inspect/build it locally:
  - `cd terraform-provider-logstruct`
  - `go build ./...`
  - Changes you make here are not committed by this repo. To contribute to the provider, commit from inside its directory and push to its own remote.

## Automated releases (gem + provider)

- Workflow: `.github/workflows/release.yml` ("Release Gem + Sync Terraform Provider").
- Triggers:
  - Push tag matching `v*` (e.g., `v0.0.1-rc1`, `v0.2.0`).
  - GitHub Release published.
  - Manual run (workflow_dispatch) with `dry_run` input.
- Behavior:
  - Builds and publishes the Ruby gem to RubyGems (requires `RUBYGEMS_API_KEY`).
  - Regenerates the provider’s embedded catalog (`scripts/export_provider_catalog.rb`), builds the provider, commits catalog changes, and tags the provider repo with the same version.
  - Enforces version alignment: the tag `vX.Y.Z` (or RC) must match `lib/log_struct/version.rb` unless run in dry-run.

## Dry-run mode

- CI dry-run lets you smoke-test the workflow without publishing anything:
  - Actions → "Release Gem + Sync Terraform Provider" → Run workflow → `dry_run=true`.
  - The workflow builds the gem and provider, shows diffs, and skips pushes/tags/uploads.
- Local dry-run for the GitHub Actions workflow isn’t practical without a runner like `act`. You can still sanity-check pieces locally:
  - `gem build logstruct.gemspec`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DocSpring/logstruct](https://github.com/DocSpring/logstruct) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
