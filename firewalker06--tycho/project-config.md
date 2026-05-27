---
trigger: always_on
description: `bin/tycho` is the executable and boots through `HQ::CLI` in `lib/hq/cli.rb`. Keep the main Bubbletea model and screen-level update flow in `lib/hq/app.rb`, config loading in `lib/hq/registry.rb`, terminal input shims in `lib/hq/bubbletea_input.rb`, domain and process-management logic under `lib/hq/domain/`, form/composer components under `lib/hq/ui/components/`, and rendering split between the aggregator in `lib/hq/ui/rendering.rb` and focused modules under `lib/hq/ui/rendering/`.
---

# Repository Guidelines

## Project Structure & Module Organization

`bin/tycho` is the executable and boots through `HQ::CLI` in `lib/hq/cli.rb`. Keep the main Bubbletea model and screen-level update flow in `lib/hq/app.rb`, config loading in `lib/hq/registry.rb`, terminal input shims in `lib/hq/bubbletea_input.rb`, domain and process-management logic under `lib/hq/domain/`, form/composer components under `lib/hq/ui/components/`, and rendering split between the aggregator in `lib/hq/ui/rendering.rb` and focused modules under `lib/hq/ui/rendering/`.

Domain files are intentionally small: `constants.rb` owns log/schema paths, `version_lookup.rb` handles RubyGems lookups, `kamal_action.rb` manages detached Kamal commands, `app_project.rb` handles project metadata and health checks, `managed_agent.rb` manages Codex/Claude-compatible execution and structured results, and `agent_store.rb` persists managed agents.

Project definitions live in `~/.tycho/config/hq.yml`, system prompt templates live in `~/.tycho/config/system_prompts.yml`, and structured managed-agent output is described by `~/.tycho/config/schemas/agent_result.json`. Project status, key decisions, and roadmap live in `docs/PROJECT_STATUS.md`; update it when durable priorities, milestones, or architectural decisions change. Research and workflow notes live under `docs/`, including `docs/GOTCHAS.md`, `docs/REMOTE_SERVER.md`, `docs/research/charm-ruby.md`, `docs/research/codex-json-schema-research.md`, and `docs/research/claude-json-schema-research.md`.

Runtime artifacts are written to `~/.tycho/logs/`, including app state files such as `actions.json` and `managed_agents.json`, application logs in `hq.log`, project logs under `~/.tycho/logs/projects/{project}/`, archived project logs under `~/.tycho/logs/projects/archived/`, and per-agent logs/status/result files under `~/.tycho/logs/agents/`. Keep automated checks under `test/`; the existing rendering regression coverage lives in `test/rendering_test.rb`.

## Build, Test, and Development Commands

- `bundle install`: install Ruby gems declared for Ruby 3.2+.
- `bin/tycho`: start the TUI locally.
- `bundle exec bin/tycho`: run through Bundler when debugging gem resolution issues.
- `bundle exec bin/tycho serve [--host 127.0.0.1] [--port 7373]`: start the local Remote Sessions JSON API and web UI for managed-agent control.
- `bundle exec bin/tycho schedule [list|daemon --once|daemon --dry-run]`: list schedules, run the scheduled-agent daemon, or run a single scheduler tick.
- `bin/test`: run the public CI-equivalent Ruby syntax and regression suite.
- `bundle exec ruby -c bin/tycho`: syntax-check the main executable before opening a PR.
- `bundle exec ruby test/registry_test.rb`: verify registry loading for split config and system prompt interpolation.
- `bundle exec ruby test/parser_test.rb`: verify synthetic Claude parser fixture shapes.
- `bundle exec ruby test/managed_agent_test.rb`: verify managed-agent execution, memory, and structured result behavior.
- `bundle exec ruby test/remote_server_test.rb`: verify Remote Sessions agent create/edit/chat/archive service behavior.
- `bundle exec ruby test/tailscale_test.rb`: verify Tailscale self-status parsing and MagicDNS URL derivation.
- `bundle exec ruby test/terminal_qr_test.rb`: verify compact terminal QR rendering for the Remote UI URL.
- `bundle exec ruby test/rendering_test.rb`: run the rendering and interaction regression checks for the TUI.
- Project status, key decisions, and roadmap are documented in `docs/PROJECT_STATUS.md`.
- Codex and Claude structured output research notes are documented in `docs/research/codex-json-schema-research.md` and `docs/research/claude-json-schema-research.md`.
- Logs, detail views, chat, and agent forms now render in-app, including sidebar views for log inspection.

If you introduce new tooling, document the command here and keep it runnable from the repo root.

## Coding Style & Naming Conventions

Follow the existing Ruby style across `bin/tycho` and `lib/hq/**/*.rb`: two-space indentation, snake_case for methods and variables, SCREAMING_SNAKE_CASE for constants, and short guard clauses where they simplify flow. Keep classes and modules focused, and prefer small helper methods over deeply nested conditionals.

Preserve the current separation of concerns: registry/config parsing stays out of the TUI layer, managed-agent and Kamal behavior belongs in domain objects, and screen/layout code should stay in the UI modules.

Preserve the current file-level conventions: `# frozen_string_literal: true`, double-quoted strings, and concise comments only where the code is not obvious.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [firewalker06/tycho](https://github.com/firewalker06/tycho) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
