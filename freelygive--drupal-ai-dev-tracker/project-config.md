---
trigger: always_on
description: - Define how ChatGPT Codex operates in this repo.
---

# AGENTS.md — Codex Operating Guide

Purpose
- Define how ChatGPT Codex operates in this repo.
- Ensure safe, repeatable starts; keep work focused and auditable.

Scope & Permissions
- Allowed edits: `AGENTS.md` and `Codex_Branch.md` only (unless the user explicitly requests code changes).
- Primary role: Review Claude’s changes, provide concise, high‑impact feedback, and capture context needed to resume work later.

Quick Reinit (Fresh Start)
- Environment:
  - `ddev start`
  - `ddev composer install`
- Install (clean DB only):
  - `ddev drush site:install --existing-config --account-pass=admin`
- Populate data:
  - Import issues: `ddev drush ai-dashboard:import-all`
  - Content sync (live→local):
    - On live: `ddev drush aid-export`
    - On local: `ddev drush aid-import` (or `--source=local`, `--replace`, `--live-url=...`)
- Dev hygiene:
  - Cache: `ddev drush cr`
  - Lint: `ddev exec vendor/bin/phpcs --standard=Drupal,DrupalPractice web/modules/custom`
  - Tests: `ddev exec vendor/bin/phpunit -c web/core/phpunit.xml.dist`

Review Workflow
- Read context files first: `CLAUDE.md`, `CLAUDE-BRANCH.md`, `README.md`.
- Inspect changes since last review: `git status`, `git diff`, and targeted diffs for `web/modules/custom/ai_dashboard/...` and `config/sync/`.
- Record findings in `Codex_Branch.md` under a “Concise Code Review” section.
- Only propose changes that materially improve correctness, safety, or maintainability.

Focus Areas (for Claude’s work)
- Drush commands: unique names, consistent PHP 8 Attributes, clear options/aliases.
- File handling: use Drupal File API (`prepareDirectory`, `file_save_data`) for `public://` paths.
- Avoid calling Drush from Drush: prefer `\Drush\Drush::process()` or service APIs for config import/export.
- Entity references: set via associative arrays (`['target_id' => $nid]`).
- Dependency Injection: prefer injected services over `\Drupal::service()`.
- Local DX: keep Twig debug/auto-reload in `development.services.yml` when appropriate.

Safety Rules
- Do not modify application code unless explicitly asked.
- Never commit secrets; prefer DDEV env files for local overrides.
- Use `ddev drush` for site ops; prefer config sync (`cex/cim`) over manual config edits.

Repository Quick Reference
- Structure: web root `web/`; custom code `web/modules/custom/`, `web/themes/custom/`; contrib under `web/modules/contrib/`, `web/themes/contrib/`; config in `config/`.
- Testing: PHPUnit with tests under each module at `tests/src/{Unit,Kernel,Functional}`; run via the core phpunit config.
- Coding style: Drupal/DrupalPractice standards; PSR‑4 under module `src/`; DI-friendly, small functions, avoid globals.

Hand-off Notes
- All Codex feedback lives in `Codex_Branch.md` for the active branch.
- If commands or behaviors appear to conflict (e.g., duplicate Drush command names), flag in `Codex_Branch.md` as “Must Fix” and wait for user approval before patching.

Drupal Code Analysis Prompts

Use these preset prompts to quickly audit Drupal codebases for quality, security, maintainability, and alignment with Drupal best practices. Tailor the file paths/module names as needed.

1) Repository Triage
- Prompt: “Scan this repo and list all custom modules, themes, services, plugins, Drush commands, routes, entities, schema files, and config schema files. For each, note file paths and primary responsibilities.”
- Goal: Build an index of moving parts before deep-diving.

2) Coding Standards & Style
- Prompt: “Assess adherence to Drupal Coding Standards and DrupalPractice: PSR‑4 autoloading, two‑space indentation, naming conventions, docblocks, and hook implementations. Flag violations and suggest fixes that align with drupal/coder.”
- Commands: `ddev exec vendor/bin/phpcs --standard=Drupal,DrupalPractice web/modules/custom`

3) Architecture & DI
- Prompt: “Evaluate architecture: controllers thin with business logic in services; dependency injection over \Drupal::static calls; use of plugins instead of switch/if trees; appropriate use of config vs content entities; queues/batch for long tasks. Provide targeted refactors only where the gain is clear.”

4) Security Review
- Prompt: “Perform a security audit: access checks on routes/controllers; correct permission requirements; CSRF protection on forms and non‑idempotent routes; XSS protection (no unsafe #markup, use placeholders and sanitization); parameterized DB queries; safe file uploads (managed files, extensions, private scheme if sensitive); avoid storing secrets in config. Report only high‑signal issues.”

5) Cacheability & Performance
- Prompt: “Review render cache metadata: ensure cache tags/contexts/max‑age are set or bubbled; avoid cache poisoning; Views caching configured; lazy builders where appropriate; avoid heavy queries in loops; verify route/controller caching suitability. Suggest concrete improvements with rationale.”

6) Database & Schema
- Prompt: “Inspect custom tables and entity storage: schema defined in hook_schema(); update hooks present and idempotent; entity schema for content/config entities; use of EntityQuery/TypedData instead of raw SQL where possible; safe migrations/updates. Highlight risky patterns and safer alternatives.”

7) Configuration Management

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FreelyGive/Drupal-AI-Dev-Tracker](https://github.com/FreelyGive/Drupal-AI-Dev-Tracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
