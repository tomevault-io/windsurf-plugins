---
trigger: always_on
description: **Purpose:** Single source of truth for all AI coding assistants (Claude Code, Cursor AI, OpenAI Codex, Google Gemini, GitHub Copilot, and others). Ensures all agents work with consistent guidelines and patterns.
---

# AGENTS.md - Documentation for AI Agents

**Purpose:** Single source of truth for all AI coding assistants (Claude Code, Cursor AI, OpenAI Codex, Google Gemini, GitHub Copilot, and others). Ensures all agents work with consistent guidelines and patterns.

> `CLAUDE.md` in the repo root is a symlink to this file. Update **only** `AGENTS.md`.
> `.claude/` is a symlink to `.agents/`. Edit files in `.agents/`.

## Detailed Documentation

**Comprehensive guides for specific tasks:**

| Category        | Guide                                                                                           | Purpose                                                                        |
| --------------- | ----------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------ |
| Product         | [Product Spec](docs/PRODUCT_SPEC.md)                                                            | Non-technical "why" and "for whom": problem, audience, capabilities, non-goals |
| Architecture    | [Architecture](docs/ARCHITECTURE.md)                                                            | Module layout, data flow, parse pipeline, emoji catalog                        |
| Technologies    | [Technologies](docs/TECHNOLOGIES.md)                                                            | Stack overview with versions and roles                                         |
| Standards       | [Standards](docs/STANDARDS.md)                                                                  | TypeScript / Biome lint + format conventions, naming, exports                  |
| Commands        | [Development Commands](docs/DEVELOPMENT_COMMANDS.md)                                            | npm scripts, Vitest runs, Vite build, watch loops                             |
| Testing         | [Testing](docs/TESTING_GUIDE.md)                                                                | Vitest setup, test conventions, regenerating expectations                      |
| Runtimes        | [Runtimes](docs/RUNTIMES.md)                                                                    | Node, browsers, ESM vs CommonJS, bundlers consuming the package                |
| Build & Deploy  | [Build & Deploy](docs/BUILD_DEPLOY.md)                                                          | Vite production bundle, npm publish, GitHub release pipeline                   |
| Emoji Providers | [Emoji Providers](docs/EMOJI_PROVIDERS.md)                                                      | Twemoji CDN, custom CDNs, shortcode dialects (Slack/GitHub/Discord)            |
| Performance     | [Performance](docs/PERFORMANCE.md)                                                              | Lookup hot paths, RegExp caches, bundle size, large catalog handling           |
| API Reference   | [API Reference](docs/API_REFERENCE.md)                                                          | Public methods, types, options, return values                                  |
| Security        | [Security](docs/SECURITY.md)                                                                    | XSS in HTML output, input validation, npm publish security, dependency hygiene |
| Documentation   | [Documentation Guide](docs/DOCUMENTATION_GUIDE.md)                                              | When and how to update docs                                                    |
| AI Agents       | [Agent Onboarding](docs/AI_AGENT_ONBOARDING.md), [Agent Collaboration](docs/AI_AGENT_COLLAB.md) | Setup, handoff, coordination                                                   |
| Forking         | [Fork Customization](docs/FORK_CUSTOMIZATION.md)                                                | Step-by-step rebrand of the package into a new product                         |
| Skills/Agents   | [.agents/README.md](.agents/README.md)                                                          | Available skills, slash commands, and subagents for this repo                  |

## Project Overview

**Universal Emoji Parser** — a TypeScript library that parses emoji **unicodes** and **shortcodes** in arbitrary text and converts them into HTML `<img>` tags backed by the [Twemoji](https://github.com/jdecked/twemoji) CDN, or between unicode and shortcode forms. The shortcode dictionary is pre-curated from [emojilib](https://github.com/muan/emojilib) and [`unicode-emoji-json`](https://www.npmjs.com/package/unicode-emoji-json) into a single static catalog (`src/lib/emoji-lib.json`) so the runtime never has to merge the two upstream sources.

The package targets:

- Twitter, GitHub, Slack, Discord, Google Chat, and Microsoft Teams shortcode dialects (single dictionary, normalized to one canonical slug per emoji)
- Both **CommonJS** (`require('universal-emoji-parser')`) and **ES modules** (`import uEmojiParser from 'universal-emoji-parser'`)
- Server-side (Node ≥ 22) and browser environments (consumers bundle via Vite/rollup/esbuild)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DailyBotHQ/universal-emoji-parser](https://github.com/DailyBotHQ/universal-emoji-parser) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
