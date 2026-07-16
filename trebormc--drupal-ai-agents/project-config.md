---
trigger: always_on
description: - **User interaction**: Respond in the same language the user writes in.
---

# Drupal AI Agents

## Language

- **User interaction**: Respond in the same language the user writes in.
- **Everything else in English**: All code, variables, functions, comments, docblocks, commit messages, and generated content must always be in English.
- **Skills and agents**: Match the user's intent to the appropriate English-defined skill by semantic meaning, not literal keyword matching.

## Design Principles

- **Simple and elegant solutions first.** Always prefer the simplest, cleanest approach. Avoid over-engineering.

## DDEV Environment

You run inside an AI container (OpenCode or Claude Code). The project uses multiple DDEV containers:

| Container | Access method | Purpose |
|-----------|---------------|---------|
| **Your container** | Direct | Agents, file access, bash |
| **Web** (`web`) | SSH | PHP, Drupal, Drush, Composer |
| **Beads** (`beads`) | `bd` wrapper | Git-backed task tracking |
| **Playwright MCP** (`$PLAYWRIGHT_MCP_URL`) | HTTP MCP | Chromium browser testing |

**All PHP/Drupal commands must use SSH:**

```bash
ssh web drush cr
ssh web ./vendor/bin/phpstan analyse $DDEV_DOCROOT/modules/custom
ssh web ./vendor/bin/phpunit $DDEV_DOCROOT/modules/custom/mymodule
```

**CRITICAL**: Always use `ssh web drush <command>` to run Drush commands (the `drush` alias is available in the web container's PATH).
**CRITICAL**: Never hardcode `web/` -- use `$DDEV_DOCROOT` (varies per project: `web/`, `docroot/`, etc.).

**Available variables:** `$DDEV_PRIMARY_URL` (HTTPS), `$DDEV_HTTP_URL` (HTTP — use for browser testing), `$DDEV_SITENAME`, `$DDEV_DOCROOT`, `$PLAYWRIGHT_MCP_URL`

## Model Strategy

Agents use model tokens that resolve to real model names at sync time:

| Token | Default | Used for |
|-------|---------|----------|
| `MODEL_MAIN` | CHEAP (OpenCode) / NORMAL (Claude Code) | The orchestrator: main conversation loop that receives prompts and delegates |
| `MODEL_GENIUS` | same as SMART | Hardest tasks: important code reviews, architecture decisions |
| `MODEL_SMART` | Opus 4.6 | Quality gates, planning, research, delegation advice |
| `MODEL_NORMAL` | Sonnet 4.5 | Development, review |
| `MODEL_CHEAP` | Haiku 4.5 | Exploration, fast tasks |
| `MODEL_APPLIER` | Haiku 4.5 | Mechanical code application |
| `MODEL_VISION` | same as NORMAL | Image/screenshot interpretation — must accept image input |

`GENIUS` falls back to `SMART`, `VISION` falls back to `NORMAL`, and `MAIN` falls back to `CHEAP` (OpenCode) / `NORMAL` (Claude Code) when not defined. Most NORMAL/CHEAP models cannot see images — anything that must interpret a screenshot goes through `MODEL_VISION` (the `visual-analyzer` agent). The orchestrator model dominates session cost, which is why `MODEL_MAIN` is deliberately cheap — quality comes from delegating to the right specialist, not from the orchestrator itself.

To change models, define only the variables you want to override (variable-level cascade, higher wins): repo `.env.agents` < `~/.ddev/agents-sync/.env.agents` on the host (all projects) < `.ddev/.env.agents` (one project).

## Agents

| Agent | Model | Purpose |
|-------|-------|---------|
| `drupal-dev` | NORMAL | Backend: modules, services, entities, plugins, APIs |
| `drupal-theme` | NORMAL | Frontend: Twig, CSS, JS, Tailwind |
| `drupal-test-generator` | NORMAL | Test generation: analyzes code, picks test type, generates tests |
| `code-explorer` | CHEAP | Context firewall for exploration: sweeps wide in its own context, returns only relevant paths + file:line pointers |
| `task-router` | SMART | Delegation advisor: decides which agent should handle a task |
| `applier` | APPLIER | Apply SEARCH/REPLACE blocks mechanically |
| `code-review` | GENIUS | Judge panel (7 perspectives incl. accessibility, translations, test coverage): PLAN mode → Implementation Contract; CODE mode → quality + plan conformance |
| `output-verifier` | SMART | Validate outputs against requirements |
| `ralph-planner` | SMART | Generate requirements.md for autonomous execution |
| `visual-test` | VISION | Playwright MCP browser automation |
| `visual-analyzer` | VISION | Interprets image/screenshot content: design review, UI analysis, comparisons |

### When to Use Each Agent

| Scenario | Agent |
|----------|-------|
| ANY project-wide search: find files, locate code, map structure, find usages | `code-explorer` (never search inline — see Delegation Protocol rule 5) |
| Backend PHP: modules, services, entities, plugins | `drupal-dev` |
| Frontend: Twig, CSS, JS, Tailwind | `drupal-theme` |
| Generate tests for Drupal code | `drupal-test-generator` |
| Plan a significant task (design consensus → Implementation Contract) | `code-review` (PLAN mode) |
| Validate code quality / conformance to the contract | `code-review` (CODE mode) |
| Validate non-code outputs (plans, configs, docs) | `output-verifier` |
| Browser screenshots and visual verification | `visual-test` |
| Interpret what a screenshot/image SHOWS (design, UI, comparisons) | `visual-analyzer` |
| Generate requirements.md for Ralph autonomous loop | `ralph-planner` |
| Apply SEARCH/REPLACE blocks mechanically | `applier` |
| Unsure which agent fits, task spans several agents, or needs decomposition | `task-router` |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [trebormc/drupal-ai-agents](https://github.com/trebormc/drupal-ai-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-16 -->
