---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

`rails-hyperdrive` is a **dev-only Rails engine gem** that mounts an MCP (Model Context Protocol) server at `/_hyperdrive/mcp` exposing 8 introspection tools for AI coding agents, plus `hyperdrive:init` / `hyperdrive:sync` generators that discover and install two artifact types — **skills** (lazy) and **guidelines** (eager) — shipped by companion gems under a documented contract, and a networked `hyperdrive:discover` that suggests uninstalled companion gems for the app's stack from rubygems.

**rails-hyperdrive is the mechanism; companion gems (`rails-hyperdrive-<library>`) are the content.** The gem ships no skills or guidelines of its own — only the contract, the discovery/install engine, and a single generated `stack.md`.

This is the gem itself, **not** an app that uses it. There is no host Rails app — specs boot a tiny in-memory app via Combustion at `spec/internal/`.

`README.md` describes the user-facing golden path.

## Commands

```bash
bin/setup                                          # bundle install
bundle exec rspec                                  # full suite (default rake task)
bundle exec rspec spec/hyperdrive/tools_spec.rb   # single file
bundle exec rspec -e "name fragment"               # filter by example name
bundle exec rspec --tag smoke                      # opt-in end-to-end smoke (slow; ~60s first run)
bin/console                                        # IRB with hyperdrive loaded
bin/bump patch|minor|major                         # bump the gem version (see Versioning)
bin/playground minimal|services|full_stack         # copy a smoke fixture app to gitignored playground/<variant>/,
                                                   # bundle it against this checkout, run hyperdrive:init — for
                                                   # manual poking (--force overwrites, --no-init skips init)

# CI matrix is Ruby {3.2, 3.3, 3.4} × Rails {7.2, 8.1}.
# Reproduce a specific slot locally:
RAILS_VERSION=7.2 bundle install && RAILS_VERSION=7.2 bundle exec rspec
```

Coverage is written to `coverage/` by SimpleCov (configured in `spec/spec_helper.rb`).

## Versioning

The gem follows [Semantic Versioning](https://semver.org). `lib/rails/hyperdrive/version.rb` is the **single source of truth** — `rails-hyperdrive.gemspec` reads `Rails::Hyperdrive::VERSION` from it, as does the `describe_app` MCP tool (`mcp_server.rb`). Never hand-edit the version anywhere else.

Bump with `bin/bump <patch|minor|major|X.Y.Z>` (or `mise run bump <level>`). The script:

1. Rewrites the `VERSION` constant in `version.rb`.
2. Rolls the `## [Unreleased]` section of `CHANGELOG.md` into a dated `## [X.Y.Z]` section and refreshes the link-reference footer.
3. Prints the suggested `git commit` + `git tag` commands. Pass `--commit` to make the `chore(release): vX.Y.Z` commit, and `--tag` to also create the annotated `vX.Y.Z` tag. Use `--dry-run` to preview without writing.

Record user-facing changes under `## [Unreleased]` in `CHANGELOG.md` as you go, so a release bump just dates and tags them.

Publishing to RubyGems is tag-triggered (`bin/bump <level> --commit --tag` then push the tag). Do **not** combine `bin/bump --tag` with `rake release` — both create the `vX.Y.Z` tag. Full release runbook is in [`RELEASING.md`](RELEASING.md).

`bin/bump` and the release tooling cover **only the root gem**. The `bundler-hyperdrive` plugin gem (below) versions independently via `bundler-hyperdrive/lib/bundler/hyperdrive/version.rb` and has no bump script, tags, or publish workflow yet.

## Architecture

### Composition root

`lib/rails/hyperdrive/mcp_server.rb` is where everything wires together. It builds a single `MCP::Server` with the 8 tools and 2 resource families, then wraps the `StreamableHTTPTransport` in `Safety::RackMiddleware` and exposes it as a Rack app. The engine's `config/routes.rb` mounts that rack app at `/mcp`. `McpServer.reset!` exists for test isolation — singletons are intentional.

### Safety model (defense in depth)

Three layers, all keyed off `Rails::Hyperdrive.dev_mode?` (the single source of truth in `lib/rails/hyperdrive.rb`):

1. **Engine load-time warning** (`engine.rb`) — loads in any env so production boots don't blow up, just logs a warning.
2. **Rack middleware** (`safety/rack_middleware.rb`) — 403s every request outside `Rails.env.development?` or with an Origin outside the allowlist (`localhost`, `127.0.0.1`, `[::1]`).
3. **Per-tool `with_dev_guard`** (`tools/base.rb`) — catches direct invocations (tests, rake tasks) that bypass the transport.

When adding new tools, always inherit from `Tools::Base` and wrap the body in `with_dev_guard { ... }`. The block also rescues and shapes exceptions into `respond_error`.

SQL safety (`sql_safety.rb`) is a regex pair: an allowed-leader pattern (`SELECT`/`WITH...SELECT`/`EXPLAIN`/`SHOW`/`PRAGMA`) plus a forbidden-token denylist (to catch a `DELETE` smuggled inside a CTE). It is a **guardrail against accidental AI damage, not a sandbox** — the user has root on their dev DB.

### Shared state between generator and runtime


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Bakaface/rails-hyperdrive](https://github.com/Bakaface/rails-hyperdrive) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-02 -->
