---
trigger: always_on
description: llmist is a streaming-first, multi-provider LLM client with a custom "gadget" tool-calling system. The project is a npm workspaces + Turborepo monorepo.
---

# Claude Code Guidelines for llmist

## Overview

llmist is a streaming-first, multi-provider LLM client with a custom "gadget" tool-calling system. The project is a npm workspaces + Turborepo monorepo.

**Packages:**
- `llmist` - Core library (npm: llmist)
- `@llmist/cli` - Command-line interface (npm: @llmist/cli)
- `@llmist/testing` - Testing utilities and mocks (npm: @llmist/testing)
- `@llmist/docs` - Documentation site (private, at llmist.dev)

## Git Workflow

```
feature-branch  →  PR to dev  →  PR to main
     ↑                              ↑
   (work)                      (release only)
```

### Rules

1. **All work on feature branches** - Never commit directly to `dev` or `main`
2. **PRs go to `dev` first** - No direct PRs to `main` from feature branches
3. **`dev` → `main` for releases** - CI validates this rule

### Workflow

```bash
# 1. Create feature branch from dev
git checkout dev && git pull origin dev
git checkout -b feat/my-feature

# 2. Work and commit (conventional commits required)
git commit -m "feat(agent): add new capability"

# 3. Push and create PR to dev
git push -u origin feat/my-feature
gh pr create --base dev

# 4. After merge to dev, create release PR
gh pr create --base main --head dev --title "chore(release): merge dev to main"
```

### Branch Sync & Recovery

After each release, `main` is automatically synced to `dev`. If sync fails, the workflow shows a failure in GitHub Actions (you'll get an email notification).

**Manual recovery if branches desync:**
```bash
git fetch origin
git checkout dev
git merge origin/main
# Resolve any conflicts
git push origin dev
```

## Commands

### Build & Test
```bash
npm install              # Install dependencies
npm run build            # Build all packages (uses Turborepo)
npm run test             # Run all tests (uses Vitest)
npm run typecheck        # Type-check all packages
npm run lint             # Lint with Biome
npm run format           # Format with Biome
npm run check            # Lint + format
```

### Package-Specific
```bash
npm run test             # Run all tests
npm run test:e2e         # E2E tests (use mocks)
npm run test:coverage    # Tests with coverage
```

### Documentation
```bash
npm run docs:dev         # Start docs dev server
npm run docs:build       # Build docs site
npm run docs:preview     # Preview built docs
```

## Project Structure

```
packages/
├── llmist/              # Core library
│   └── src/
│       ├── agent/       # Agent, builder, stream processor, hooks, compaction
│       ├── core/        # LLMist client, messages, execution tree, models
│       ├── gadgets/     # Parser, executor, registry, helpers, exceptions
│       ├── skills/      # Agent Skills standard (SKILL.md parser, registry, activation)
│       ├── providers/   # Anthropic, OpenAI, Gemini adapters
│       ├── utils/       # Formatting, timing, config resolution
│       ├── logging/     # tslog-based logging
│       ├── session/     # Session management
│       └── e2e/         # End-to-end tests
├── cli/                 # CLI application
│   └── src/
│       ├── agent-command.ts    # Main agent command
│       ├── complete-command.ts # Completion command
│       ├── config.ts           # TOML config parsing
│       ├── skills/             # Skill CLI commands, config, slash handler
│       └── tui/                # Terminal UI
├── testing/             # Testing utilities
│   └── src/
│       ├── mock-builder.ts     # Fluent mock API
│       ├── mock-adapter.ts     # Provider mock
│       ├── gadget-testing.ts   # testGadget() utility
│       └── helpers.ts          # Test helpers
└── docs/                # Astro Starlight docs site
    └── src/content/docs/
        ├── library/     # Core library docs
        ├── cli/         # CLI docs
        ├── testing/     # Testing docs
        └── reference/   # API reference

examples/                # Runnable examples (01-32)
└── gadgets/             # Example gadgets (calculator, filesystem, etc.)
```

## Key Concepts

### Gadgets
Tools that LLMs can call. Two styles:
```typescript
// Class-based (recommended for complex gadgets)
class MyGadget extends Gadget({
  description: 'Does something',
  schema: z.object({ param: z.string() }),
}) {
  execute(params: this['params']): string { ... }
}

// Function-based (simple one-offs)
const myGadget = createGadget({
  name: 'MyGadget',
  schema: z.object({ param: z.string() }),
  execute: (params) => { ... },
});
```

### Skills
Markdown-based instruction packages following the [Agent Skills open standard](https://agentskills.io). Skills extend agent capabilities through prompt injection and context management, not code execution. Three-tier progressive disclosure manages context budget:
- **Tier 1** (~100 tokens) - Name + description, always loaded
- **Tier 2** (<5K tokens) - Full SKILL.md body, loaded on activation
- **Tier 3** (unlimited) - Scripts, references, assets, loaded on demand

```typescript
// Load skills from standard locations
const registry = discoverSkills({ projectDir: process.cwd() });

// Or create programmatically
const skill = Skill.fromContent(`---
name: code-review
description: Review code for bugs and best practices

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zbigniewsobiecki/llmist](https://github.com/zbigniewsobiecki/llmist) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
