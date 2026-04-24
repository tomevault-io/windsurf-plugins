---
trigger: always_on
description: This is the **uniswap-ai** monorepo providing Uniswap-specific AI tools (skills, plugins, agents) for developers and AI agents integrating the Uniswap ecosystem.
---

# CLAUDE.md - Uniswap AI Project Guidelines

## Overview

This is the **uniswap-ai** monorepo providing Uniswap-specific AI tools (skills, plugins, agents) for developers and AI agents integrating the Uniswap ecosystem.

## Core Requirements

### Nx Usage

- **REQUIREMENT**: Use Nx for ALL packages and tooling in this monorepo
- Every package must be an Nx project with proper configuration
- Use Nx generators, executors, and workspace features wherever possible
- Leverage Nx's dependency graph and caching capabilities

### Package Structure

- All packages must be properly configured Nx libraries or applications
- Use Nx's project.json for configuration
- Follow Nx best practices for monorepo organization

### Development Workflow

- Use Nx commands for all operations (build, test, lint, etc.)
- Maintain proper inter-package dependencies through Nx
- Ensure all packages are part of the Nx workspace graph

### Code Quality Enforcement

After making any code changes, Claude Code MUST:

1. **Format the code**: Run `npx nx format:write --uncommitted` to format all uncommitted files
2. **Lint the code**: Run `npx nx affected --target=lint --base=HEAD~1` to check for linting errors
3. **Typecheck the code**: Run `npx nx affected --target=typecheck --base=HEAD~1` to typecheck affected projects
4. **Lint markdown files**: Run `npm exec markdownlint-cli2 -- --fix "**/*.md"`
5. **Lint documentation prose**: Run `npm run docs:lint` to check documentation quality with Vale

## Package Scopes

| Type    | Scope      | npm | Marketplace                   |
| ------- | ---------- | --- | ----------------------------- |
| Plugins | `@uniswap` | No  | Yes (Claude Code Marketplace) |

## Repository Structure

```text
uniswap-ai/
├── .github/
│   ├── workflows/           # CI/CD workflows
│   ├── actions/             # Reusable composite actions
│   └── scripts/             # CI scripts
├── .claude-plugin/
│   └── marketplace.json     # Claude Code marketplace config
├── .claude/
│   └── rules/               # Agent rules (agnostic design)
├── docs/                    # VitePress documentation
├── evals/                   # AI tool evaluations (Promptfoo)
│   ├── rubrics/             # Shared evaluation rubrics
│   ├── scripts/             # Custom providers and utilities
│   ├── suites/              # Per-skill eval suites
│   └── templates/           # Templates for new suites
├── packages/
│   └── plugins/             # Claude Code plugins
│       ├── uniswap-cca/     # Continuous Clearing Auction (CCA) plugin
│       ├── uniswap-driver/  # Swap & liquidity deep link planning
│       ├── uniswap-hooks/   # Uniswap v4 hooks plugin
│       ├── uniswap-trading/ # Uniswap swap integration
│       └── uniswap-viem/    # EVM blockchain integration (viem/wagmi)
├── scripts/                 # Build/validation scripts
├── nx.json
├── package.json
├── tsconfig.base.json
├── CLAUDE.md                # This file
├── AGENTS.md -> CLAUDE.md   # Symlink for agent-agnostic access
├── LICENSE                  # MIT
└── README.md
```

### Plugin Architecture

Plugins are stored in `./packages/plugins/<plugin-name>/`:

- Each plugin is a self-contained Nx package with `package.json`, `project.json`, and `.claude-plugin/plugin.json`
- The `.claude-plugin/marketplace.json` references plugins via relative paths
- Plugin validation: `node scripts/validate-plugin.cjs packages/plugins/<plugin-name>`

### Plugin Versioning

All plugins follow semantic versioning (semver):

- **Patch (1.0.X)**: Bug fixes, minor documentation updates
- **Minor (1.X.0)**: New skills, agents, or commands (backward compatible)
- **Major (X.0.0)**: Breaking changes, significant restructuring

After making any changes to `packages/plugins/`, bump the plugin version in `.claude-plugin/plugin.json`.

## Agent-Agnostic Design

All AI tools in this repo should be usable by ANY LLM coding agent, not just Claude Code:

1. **Documentation**: Use AGENTS.md (symlink to CLAUDE.md) as standard
2. **Prompts**: Write prompts that work across models (avoid Claude-specific features unless necessary)
3. **Skills**: Structure skills as markdown that any agent can interpret
4. **No vendor lock-in**: Prefer standards over proprietary features
5. **Testing**: Evals should work with multiple LLM backends

## Evals Framework

Evals are to AI tools what tests are to traditional code. This project uses [Promptfoo](https://github.com/promptfoo/promptfoo) for declarative, CI-integrated evaluations.

### Structure

```text
evals/
├── promptfoo.yaml          # Root config with default providers
├── rubrics/                # Shared evaluation rubrics (.txt files)
│   └── security-checklist.txt
├── scripts/
│   └── anthropic-provider.ts  # Custom provider for OAuth support
├── suites/                 # Per-skill eval suites
│   └── <skill-name>/
│       ├── promptfoo.yaml  # Suite-specific config
│       ├── cases/          # Test case prompts (markdown)
│       └── rubrics/        # Skill-specific rubrics (.txt files)
└── templates/              # Templates for new suites
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Uniswap/uniswap-ai](https://github.com/Uniswap/uniswap-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
