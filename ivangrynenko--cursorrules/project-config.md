---
trigger: always_on
description: This document explains how to use the rules in this repository with Cursor and serves as a single entry point that references the existing rule files. It avoids duplication by linking directly to the `.cursor/rules/*.mdc` sources.
---

# Cursor Agents Guide (Using Cursor Rules)

This document explains how to use the rules in this repository with Cursor and serves as a single entry point that references the existing rule files. It avoids duplication by linking directly to the `.cursor/rules/*.mdc` sources.

If you installed these rules via the installer, a project‑local AGENTS.md can be generated that lists only the rules you chose. By default, the installer writes AGENTS.md if absent; it overwrites only when you pass `--yes`.

## How To Use With Cursor
- Open your project in Cursor. Rules under `.cursor/rules` are discovered automatically by Cursor.
- Keep this AGENTS.md handy as your quick index to the rule set.
- For installation methods and advanced options, see `README.md`.

## Installation Options
For full installation details and examples, see `README.md`.
- Core rules only: `--core`
- Web stack (includes core): `--web-stack` or `--ws`
- Python (includes core): `--python`
- JavaScript security (includes core): `--javascript`
- All rules: `--all`
- Tag-based selection: `--tags "<expression>"` or `--tag-preset <name>`
- Ignore files control: `--ignore-files yes|no|ask`

Tag taxonomy is documented in `TAG_STANDARDS.md`.

## Rule Bundles (Source of Truth)
Below are the rule bundles and their rule files. Each item links directly to the authoritative file under `.cursor/rules/`.

### Core
- [.cursor/rules/cursor-rules.mdc](.cursor/rules/cursor-rules.mdc)
- [.cursor/rules/git-commit-standards.mdc](.cursor/rules/git-commit-standards.mdc)
- [.cursor/rules/github-actions-standards.mdc](.cursor/rules/github-actions-standards.mdc)
- [.cursor/rules/improve-cursorrules-efficiency.mdc](.cursor/rules/improve-cursorrules-efficiency.mdc)
- [.cursor/rules/pull-request-changelist-instructions.mdc](.cursor/rules/pull-request-changelist-instructions.mdc)
- [.cursor/rules/readme-maintenance-standards.mdc](.cursor/rules/readme-maintenance-standards.mdc)
- [.cursor/rules/testing-guidelines.mdc](.cursor/rules/testing-guidelines.mdc)
 - [.cursor/rules/confluence-editing-standards.mdc](.cursor/rules/confluence-editing-standards.mdc)

### Web Stack
- [.cursor/rules/accessibility-standards.mdc](.cursor/rules/accessibility-standards.mdc)
- [.cursor/rules/api-standards.mdc](.cursor/rules/api-standards.mdc)
- [.cursor/rules/build-optimization.mdc](.cursor/rules/build-optimization.mdc)
- [.cursor/rules/code-generation-standards.mdc](.cursor/rules/code-generation-standards.mdc)
- [.cursor/rules/debugging-standards.mdc](.cursor/rules/debugging-standards.mdc)
- [.cursor/rules/docker-compose-standards.mdc](.cursor/rules/docker-compose-standards.mdc)
- [.cursor/rules/drupal-authentication-failures.mdc](.cursor/rules/drupal-authentication-failures.mdc)
- [.cursor/rules/drupal-broken-access-control.mdc](.cursor/rules/drupal-broken-access-control.mdc)
- [.cursor/rules/drupal-cryptographic-failures.mdc](.cursor/rules/drupal-cryptographic-failures.mdc)
- [.cursor/rules/drupal-database-standards.mdc](.cursor/rules/drupal-database-standards.mdc)
- [.cursor/rules/drupal-file-permissions.mdc](.cursor/rules/drupal-file-permissions.mdc)
- [.cursor/rules/drupal-injection.mdc](.cursor/rules/drupal-injection.mdc)
- [.cursor/rules/drupal-insecure-design.mdc](.cursor/rules/drupal-insecure-design.mdc)
- [.cursor/rules/drupal-integrity-failures.mdc](.cursor/rules/drupal-integrity-failures.mdc)
- [.cursor/rules/drupal-logging-failures.mdc](.cursor/rules/drupal-logging-failures.mdc)
- [.cursor/rules/drupal-security-misconfiguration.mdc](.cursor/rules/drupal-security-misconfiguration.mdc)
- [.cursor/rules/drupal-ssrf.mdc](.cursor/rules/drupal-ssrf.mdc)
- [.cursor/rules/drupal-vulnerable-components.mdc](.cursor/rules/drupal-vulnerable-components.mdc)
- [.cursor/rules/generic_bash_style.mdc](.cursor/rules/generic_bash_style.mdc)
- [.cursor/rules/javascript-performance.mdc](.cursor/rules/javascript-performance.mdc)
- [.cursor/rules/javascript-standards.mdc](.cursor/rules/javascript-standards.mdc)
- [.cursor/rules/lagoon-docker-compose-standards.mdc](.cursor/rules/lagoon-docker-compose-standards.mdc)
- [.cursor/rules/lagoon-yml-standards.mdc](.cursor/rules/lagoon-yml-standards.mdc)
- [.cursor/rules/multi-agent-coordination.mdc](.cursor/rules/multi-agent-coordination.mdc)
- [.cursor/rules/node-dependencies.mdc](.cursor/rules/node-dependencies.mdc)
- [.cursor/rules/php-drupal-best-practices.mdc](.cursor/rules/php-drupal-best-practices.mdc)
- [.cursor/rules/php-drupal-development-standards.mdc](.cursor/rules/php-drupal-development-standards.mdc)
- [.cursor/rules/php-memory-optimisation.mdc](.cursor/rules/php-memory-optimisation.mdc)
- [.cursor/rules/project-definition-template.mdc](.cursor/rules/project-definition-template.mdc)
- [.cursor/rules/react-patterns.mdc](.cursor/rules/react-patterns.mdc)
- [.cursor/rules/security-practices.mdc](.cursor/rules/security-practices.mdc)
- [.cursor/rules/secret-detection.mdc](.cursor/rules/secret-detection.mdc)
- [.cursor/rules/tailwind-standards.mdc](.cursor/rules/tailwind-standards.mdc)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ivangrynenko/cursorrules](https://github.com/ivangrynenko/cursorrules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
