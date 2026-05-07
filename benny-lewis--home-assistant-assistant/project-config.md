---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Claude Code plugin for Home Assistant. It allows users to manage Home Assistant configurations through natural language—creating automations, scripts, scenes, and dashboards by describing what they want instead of manually editing YAML.

**Type**: Claude Code plugin (markdown-based, no build system or compiled code)

## Safety Invariants

All generated YAML and commands enforce eight safety invariants (canonical wording in `references/safety-invariants.md`):

1. **No unsupported attributes** - Always check `supported_features`/`supported_color_modes` before suggesting device attributes
2. **No semantic substitution** - Never replace "after no motion" (inactivity) with raw timers
3. **AST editing only** - No brittle string replacement; use Edit tool with precise old/new strings
4. **No secrets printed** - Never echo tokens; show "TOKEN is set" not the value
5. **Never deploy unless explicitly requested** - All side-effectful skills require explicit user request and confirmation
6. **Evidence tables** - All validation outputs show "what ran vs skipped"
7. **Minimal edits only** - Make only the specific changes requested; do not reorganize adjacent content
8. **Verify after config edits** - Offer deploy/reload after YAML changes; validate entity IDs exist before use

## Plugin Architecture

The plugin follows Claude Code's plugin structure:

```
.claude-plugin/
  plugin.json               # Plugin manifest - metadata, component discovery
skills/
  ha-automations/           # Automation creation + domain knowledge (user-invocable)
  ha-scripts/               # Script creation + domain knowledge (user-invocable)
  ha-scenes/                # Scene creation + domain knowledge (user-invocable)
  ha-config/                # Config organization knowledge (user-invocable)
  ha-jinja/                 # Jinja templating knowledge (user-invocable)
  ha-lovelace/              # Dashboard design knowledge (user-invocable)
  ha-naming/                # Naming conventions + audit + plan (user-invocable)
  ha-apply-naming/          # Naming execution (user-invocable, NO model invocation)
  ha-devices/               # Device knowledge + new device workflow (user-invocable)
  ha-troubleshooting/       # Debugging knowledge (user-invocable)
  ha-onboard/               # Setup wizard + connection + settings (user-invocable)
  ha-deploy/                # Deploy + rollback (user-invocable, in-skill confirmation gates)
  ha-validate/              # Validation workflow + procedures (user-invocable, agent-preloadable)
  ha-analyze/               # Setup analysis + recommendations (user-invocable)
  ha-resolver/              # Entity resolution (NOT user-invocable, agent-preloaded)
agents/
  *.md                      # 6 subagents (config-debugger, ha-config-validator,
                            # device-advisor, naming-analyzer, ha-entity-resolver,
                            # ha-log-analyzer)
helpers/
  area-search.py            # Area-based entity search (registry cross-referencing)
  entity-registry.py        # Entity registry operations
  ha-overview.py            # HA setup overview generation
  trace-fetch.py            # Automation trace fetching
  lovelace-dashboard.py     # Lovelace dashboard fetch/save/verify/find-entities
hooks/
  hooks.json                # Event-driven hooks (SessionStart, PreToolUse, PostToolUse)
  session-check.sh          # Async env check (HASS_TOKEN, HASS_SERVER, python detection)
  env-guard.sh              # PreToolUse guard for Bash commands
  docs-check.sh             # Documentation validation
  docs-check.py             # Documentation validation helper
references/
  safety-invariants.md      # Core safety rules referenced by all skills
  settings-schema.md        # Settings file schema
  hass-cli.md               # hass-cli usage reference
  ha-web-ui.md              # HA web UI reference
  dashboard-api.md          # WebSocket API contract for storage dashboards
templates/
  templates.md              # Reference templates for generated configs
```

**15 skills total:** 14 user-invocable + 1 infrastructure (ha-resolver). ha-validate is both user-invocable and agent-preloadable.

**Progressive disclosure:** Skills with long procedural content keep SKILL.md as a tight triggering surface (frontmatter, safety banner, decision rules, workflow index) and move step-by-step detail into per-skill `references/*.md`. Skills currently using this pattern: ha-apply-naming, ha-automations, ha-devices, ha-lovelace, ha-naming, ha-onboard, ha-resolver, ha-scenes, ha-scripts, ha-troubleshooting. When exploring a skill, read its SKILL.md first, then follow the Workflow Index to the specific reference file needed.

## Marketplace Packaging Note

For this repo's self-hosted single-plugin marketplace, keep `.claude-plugin/marketplace.json` pointing the plugin entry at `source: "./"`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Benny-Lewis/home-assistant-assistant](https://github.com/Benny-Lewis/home-assistant-assistant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
