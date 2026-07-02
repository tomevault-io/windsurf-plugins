---
trigger: always_on
description: **STOP. Before any tool use, classify this request using the table below.**
---

## Agent Dispatch (MANDATORY)

**STOP. Before any tool use, classify this request using the table below.**

| Request type | Action |
|---|---|
| Any `.php` file in `app/` (Action, Service, Controller, Model…) | **Dispatch to pipeline** |
| Any `.vue` file or Inertia page | **Dispatch to pipeline** |
| Migration, route, Policy, Gate, Form Request | **Dispatch to pipeline** |
| Bug investigation + fix | **Dispatch to pipeline** |
| Touches >2 files of any kind | **Dispatch to pipeline** |
| Single-line typo or comment fix | Handle directly |
| Single config key (`.env`, one key in `config/*.php`) | Handle directly |
| Documentation only (`docs/**`, `README.md`) | Handle directly (use `docs-writer` if non-trivial) |
| Changes to `.claude/` infrastructure itself | Handle directly |

**If pipeline applies → dispatch via `Agent` tool immediately. Do NOT `Read`/`Grep`/`Bash` before dispatch.**
**If ambiguous → ask exactly ONE clarifying question, then dispatch.**

Full pipeline details: `.claude/rules/workflow.md`. Run independent pipeline steps in parallel. Never ask the user which agent to use — decide autonomously.

Available agents: `ba`, `developer`, `frontend`, `tester`, `qa`, `reviewer`, `debugger`, `security-scanner`, `dba`, `ddd-architect`, `devil`, `filament`, `devops`, `integration-architect`, `laravel-refactoring-expert`, `queue-specialist`, `docs-writer`

## Claude-Specific Behavior

- Use available Skills for Laravel code style, testing, architecture, Inertia, DevOps
- If a Skill applies, prefer it over repeating rules here

## IMPORTANT

1. Before starting any task, evaluate pipeline trigger conditions in `.claude/rules/workflow.md`. If pipeline applies — start it immediately.
2. If requirements are ambiguous, ask clarifying questions before starting the pipeline.
3. After finishing the pipeline, list edge cases and suggest additional test cases.
4. If a task requires changes to more than 3 files, stop and break it into smaller tasks.
5. When there's a bug, start by writing a test that reproduces it, then fix it.
6. Every time I correct you, reflect on what went wrong and plan to prevent it.

## Setup

See `docs/SETUP.md` for system requirements, Docker setup, and common commands.

===

<laravel-boost-guidelines>
=== foundation rules ===

# Laravel Boost Guidelines

The Laravel Boost guidelines are specifically curated by Laravel maintainers for this application. These guidelines should be followed closely to ensure the best experience when building Laravel applications.

## Foundational Context

This application is a Laravel application and its main Laravel ecosystems package & versions are below. You are an expert with them all. Ensure you abide by these specific packages & versions.

- php - 8.5
- filament/filament (FILAMENT) - v5
- inertiajs/inertia-laravel (INERTIA_LARAVEL) - v3
- laravel/framework (LARAVEL) - v13
- laravel/octane (OCTANE) - v2
- laravel/prompts (PROMPTS) - v0
- laravel/pulse (PULSE) - v1
- laravel/reverb (REVERB) - v1
- laravel/socialite (SOCIALITE) - v5
- livewire/livewire (LIVEWIRE) - v4
- tightenco/ziggy (ZIGGY) - v2
- larastan/larastan (LARASTAN) - v3
- laravel/boost (BOOST) - v2
- laravel/mcp (MCP) - v0
- laravel/pail (PAIL) - v1
- laravel/pint (PINT) - v1
- laravel/telescope (TELESCOPE) - v5
- pestphp/pest (PEST) - v4
- phpunit/phpunit (PHPUNIT) - v12
- rector/rector (RECTOR) - v2
- @inertiajs/vue3 (INERTIA_VUE) - v3
- eslint (ESLINT) - v10
- laravel-echo (ECHO) - v2
- prettier (PRETTIER) - v3
- tailwindcss (TAILWINDCSS) - v4
- vue (VUE) - v3

## Skills Activation

This project has domain-specific skills available in `**/skills/**`. You MUST activate the relevant skill whenever you work in that domain—don't wait until you're stuck.

## Conventions

- You must follow all existing code conventions used in this application. When creating or editing a file, check sibling files for the correct structure, approach, and naming.
- Use descriptive names for variables and methods. For example, `isRegisteredForDiscounts`, not `discount()`.
- Check for existing components to reuse before writing a new one.

## Verification Scripts

- Do not create verification scripts or tinker when tests cover that functionality and prove they work. Unit and feature tests are more important.

## Application Structure & Architecture

- Stick to existing directory structure; don't create new base folders without approval.
- Do not change the application's dependencies without approval.

## Frontend Bundling

- If the user doesn't see a frontend change reflected in the UI, it could mean they need to run `yarn run build`, `yarn run dev`, or `composer run dev`. Ask them.

## Documentation Files

- You must only create documentation files if explicitly requested by the user.

## Replies

- Be concise in your explanations - focus on what's important rather than explaining obvious details.

=== boost rules ===

# Laravel Boost

## Artisan

- Run Artisan commands directly via the command line (e.g., `php artisan route:list`). Use `php artisan list` to discover available commands and `php artisan [command] --help` to check parameters.
- Inspect routes with `php artisan route:list`. Filter with: `--method=GET`, `--name=users`, `--path=api`, `--except-vendor`, `--only-vendor`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Mentor-Wizard/mentor-wizard-webapp](https://github.com/Mentor-Wizard/mentor-wizard-webapp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-02 -->
