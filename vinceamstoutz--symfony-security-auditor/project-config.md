---
trigger: always_on
description: Update the relevant section in the same commit/PR whenever the project evolves.
---

# CLAUDE.md

## Keeping This File Up to Date

Update the relevant section in the same commit/PR whenever the project evolves.
Rules scoped to specific paths live in `.claude/rules/` — update them too. Never
leave this file describing a state that no longer exists.

---

## Project Overview

**symfony-security-auditor** — AI-powered multi-agent security auditor for
Symfony applications. Distributed as a **Symfony bundle** (`symfony-bundle`
package type). Uses a dual-agent attacker/reviewer loop backed by `symfony/ai`
to detect vulnerabilities and produce structured reports.

> Always check `composer.json` for authoritative dependency versions — never
> rely on version numbers written here.

## Tech Stack

| Layer             | Technology                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| ----------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Language          | PHP (see `composer.json` → `require.php`)                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| Framework         | Symfony (see `composer.json`)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| LLM               | symfony/ai (provider-agnostic: Anthropic, OpenAI, Mistral, Ollama, …)                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| Packaging         | symfony-bundle + Flex recipe; standalone self-contained native binary (`box` + `static-php-cli` micro) for Linux/macOS/Windows                                                                                                                                                                                                                                                                                                                                                                                       |
| Tests             | PHPUnit (Unit / Integration / EndToEnd); 100% line coverage enforced via the custom `MinimumLineCoverageExtension` (`tools/PHPUnit/`); the report-only `ergebnis/phpunit-slow-test-detector` surfaces tests over its `maximum-duration` (500 ms), and `SlowTestGuardExtension` (`tools/PHPUnit/`) reuses that same threshold — read straight from the detector's config, single source of truth — to fail the run when a test exceeds it (per-test `#[MaximumDuration]` overrides), so genuinely slow tests break CI |
| Mutation          | Infection (100% MSI required)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vinceAmstoutz/symfony-security-auditor](https://github.com/vinceAmstoutz/symfony-security-auditor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
