---
trigger: always_on
description: - Language: Nix (NixOS configuration), YAML (Home Assistant), Jinja2 (templates)
---

# Code Review Instructions

## Project Context

**Stack:**

- Language: Nix (NixOS configuration), YAML (Home Assistant), Jinja2 (templates)
- Architecture: Declarative GitOps with immutable infrastructure
- Build System: Nix Flakes with locked dependencies
- Key Frameworks: NixOS 24.11, Home Assistant, Wyoming Protocol (Whisper STT, Piper TTS)

**Core Modules:**

- `hosts/homelab/`: System configuration (boot, networking, SSH, Comin)
- `hosts/homelab/home-assistant/`: HA service, voice processing, intents, automations
- `tests/`: Configuration validation (automation IDs, entities, services, YAML schema)
- `custom_sentences/pl/`: Polish voice command sentence patterns

**Conventions:**

- Automation IDs: `snake_case` (e.g., `startup_notification`)
- Automation aliases: `Category - Description` (e.g., `System - Startup notification`)
- Entity names: Polish lowercase, spaces → underscores (e.g., `light.salon`)
- Intent names: PascalCase (e.g., `TurnOnLight`)
- Entity ID normalization: `{{ slots.name | lower | replace(' ', '_') }}`
- Formatting: `alejandra` for Nix, 2-space indent for YAML
- Deployment: Git commit → Comin auto-pull (~60s) → nixos-rebuild
- Documentation: Inline for complex Nix expressions, Polish voice patterns

**Critical Areas (Extra Scrutiny):**

- Voice command intent matching (Polish language patterns)
- Automation logic (service calls, entity IDs, domain compatibility)
- GitOps integrity (immutable config, no config drift)
- Secrets handling (must NOT commit .secret, .key, secrets.yaml)
- Home Assistant configuration validation (test coverage required)

---

## Review Before CI Completes

You review PRs immediately, before CI finishes. Do NOT flag issues that CI will catch.

**CI Already Checks:**

- Nix code formatting (`alejandra`)
- YAML syntax and linting (`yamllint`)
- Markdown linting (`markdownlint-cli2`)
- GitHub Actions workflow linting (`actionlint`)
- Configuration validation (`nix flake check`)
- Schema validation (intent structure, Jinja2, HA services)

---

## Review Priority Levels

### 🔴 CRITICAL (Must Block PR)

**Security Vulnerabilities** (95%+ confidence)

- [ ] Secrets in code (SSH keys, API tokens, passwords in .nix files)
- [ ] Secrets files not in .gitignore (.secret, .key, secrets.yaml)
- [ ] SSH configuration allows password auth or root login
- [ ] Home Assistant exposed without authentication
- [ ] Hardcoded credentials in voice intents or automations
- [ ] Sensitive data in logs or automation descriptions

**Correctness Issues** (90%+ confidence)

- [ ] Invalid entity IDs (wrong domain.name format)
- [ ] Service calls to non-existent services
- [ ] Domain mismatch (e.g., `light.turn_on` with `switch.` entity)
- [ ] Duplicate automation IDs (causes HA conflicts)
- [ ] Breaking changes to voice intents without updating custom_sentences
- [ ] Unbalanced Jinja2 delimiters ({{ }}, {% %})
- [ ] Invalid YAML structure in custom_sentences/pl/
- [ ] GitOps drift (manual changes not in Git)
- [ ] Flake.lock not updated after input changes

### 🟡 HIGH (Request Changes)

**Maintainability** (80%+ confidence)

- [ ] Voice intents without corresponding custom_sentences patterns
- [ ] New automations without test coverage in config-validation.nix
- [ ] Complex Nix expressions without comments
- [ ] Polish voice patterns missing verb alternatives or skip words
- [ ] Service actions without state feedback (silent failures)
- [ ] Entity names not following Polish lowercase convention
- [ ] Automation aliases not following "Category - Description" format
- [ ] New Home Assistant modules without documentation

**Architecture** (75%+ confidence)

- [ ] GUI-based Home Assistant config (should be declarative in .nix)
- [ ] Hardcoded entity IDs (should use input helpers or templates)
- [ ] Duplicated automation logic (should extract to scripts)
- [ ] Direct service calls instead of intent scripts
- [ ] Missing Polish entity name normalization filter
- [ ] Tight coupling between intents and specific devices
- [ ] Over-engineering simple automations

### 🟢 MEDIUM (Suggest/Comment)

**Performance** (70%+ confidence)

- [ ] Heavy Whisper STT model without justification (current: small)
- [ ] Unnecessary automation triggers (too frequent polling)
- [ ] Large custom_sentences patterns (increases intent processing time)
- [ ] Blocking operations in automation scripts

**Best Practices** (65%+ confidence)

- [ ] Missing error handling in voice intent responses
- [ ] No TTS feedback for failed actions
- [ ] Entity IDs not descriptive enough
- [ ] Automations without conditions (always execute)
- [ ] Missing input validation in voice slot values

### ⚪ LOW (Optional/Skip)

Don't comment on:

- Personal naming style (if follows conventions)
- Minor optimizations with no measurable impact
- Alternative Nix expression styles (if valid)
- Refactoring unrelated to the change
- Anything below confidence threshold

---

## Security Deep Dive

### Secrets Management

- [ ] NO secrets in .nix, .yaml, or .md files
- [ ] Secrets referenced via environment variables or external files
- [ ] .gitignore includes: `*.secret`, `*.key`, `secrets.yaml`, `.env`
- [ ] Consider sops-nix or agenix for encrypted secrets in Git

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Automaat/klaudiusz-smart-home](https://github.com/Automaat/klaudiusz-smart-home) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-29 -->
