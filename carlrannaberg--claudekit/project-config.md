---
trigger: always_on
description: Validates all hyperlinks in markdown files to ensure they are accessible.
---

This file provides guidance to AI coding assistants working in this repository.

**Note:** CLAUDE.md is a symlink to AGENTS.md in this project.

# claudekit

A toolkit of custom commands, hooks, and utilities for Claude Code. This project provides powerful development workflow tools including git checkpointing, automated code quality checks, specification generation, and AI assistant configuration management. The toolkit is designed to be project-agnostic and enhances Claude Code functionality through shell scripts, commands, and hooks.

## Navigating the Codebase

Use the `codebase-map` tool to understand and navigate the project structure:

**Recommended formats:**
- `dsl` - For understanding code relationships, imports/exports, functions, and classes
- `tree` - For exploring directory structure and finding files

**Example usage:**
```bash
# Get code overview with DSL format (shows functions, classes, imports)
codebase-map format --format dsl

# Get directory/file structure with tree format
codebase-map format --format tree

# Focus on specific areas
codebase-map format --format dsl --include "cli/**" --exclude "**/*.test.ts"
```

The codebase map is automatically provided at the start of each session and can be configured in `.claudekit/config.json` to filter what's shown.

**For complete setup and configuration details, see the [Codebase Map Guide](docs/guides/codebase-map.md).**

## Build & Commands

This is a TypeScript-based toolkit. Key commands:

- **Install**: `npm install -g claudekit` - Install claudekit globally
- **Setup**: `claudekit setup` - Initialize claudekit in your project
- **Build**: `npm run build` - **IMPORTANT: Run after any code changes to compile TypeScript**
- **Symlinks**: `npm run symlinks` - Create/update symlinks from `.claude/` to `src/` for development
- **Test hooks**: Manually trigger by editing files or using Claude Code
- **Check shell syntax**: `bash -n script.sh`
- **Validate JSON**: `jq . settings.json`

**Important**: After making any changes to TypeScript files (hooks, commands, or library code), you MUST run `npm run build` before testing. The project uses compiled JavaScript from the `dist/` directory, not the source TypeScript files.

### Creating New Components

- **Subagents**: See [Subagent Development Guide](docs/guides/creating-subagents.md) for creating AI assistant subagents
- **Commands**: Use `/create-command` in Claude Code for guidance
- **Hooks**: Follow self-contained patterns in `src/hooks/`

## Using Subagents

This project includes specialized subagents in `.claude/agents/` that provide deep expertise in specific technical domains. You should proactively use the Task tool to delegate to these subagents when working on relevant tasks.

### ⚠️ MANDATORY REQUIREMENT: Use Specialized Subagents

**ALL tasks and issues MUST be handled by specialized subagents.** Do not attempt to solve problems directly:

1. **For any technical issue**: Use `triage-expert` first to diagnose and route to the appropriate specialist
2. **For domain-specific work**: Delegate directly to the relevant expert (typescript-expert, react-expert, etc.)
3. **For code review**: Always use `code-review-expert` for comprehensive analysis
4. **For complex debugging**: Route through `triage-expert` to identify the right specialist
5. **For searching code content**: Use `code-search` agent to find specific implementations, string literals, or patterns within files

**Examples of REQUIRED delegation:**
- Searching for specific code implementations or content → Use `code-search`
- TypeScript build/compilation errors → Use `typescript-build-expert`
- Advanced type system issues (generics, conditionals) → Use `typescript-type-expert`
- General TypeScript problems → Use `typescript-expert`
- Vitest configuration and framework issues → Use `vitest-testing-expert`
- Test strategy and coverage issues → Use `testing-expert`
- Code smells and refactoring → Use `refactoring-expert`
- CLI command issues → Use `cli-expert`
- Git workflow issues → Use `git-expert`
- Code quality and linting → Use `linting-expert`
- GitHub Actions CI/CD → Use `github-actions-expert`
- Shell script problems → Use `devops-expert`
- Node.js runtime issues → Use `nodejs-expert`

**Never attempt to:**
- Debug complex issues without specialist expertise
- Make architectural decisions without domain expert input
- Implement solutions in unfamiliar domains without consulting experts
- Skip the triage process for unclear problems

### Available Subagents

**Build Tools:**
- `webpack-expert` - Webpack configuration, bundle optimization, plugins/loaders
- `vite-expert` - Vite development, ESM patterns, HMR optimization

**TypeScript:**
- `typescript-expert` - General TypeScript/JavaScript expertise
- `typescript-build-expert` - Compiler configuration, module resolution
- `typescript-type-expert` - Advanced type system, generics, conditional types

**React & Frontend:**
- `react-expert` - React components, hooks, patterns
- `react-performance-expert` - React optimization, DevTools Profiler, memoization
- `css-styling-expert` - CSS architecture, responsive design, CSS-in-JS
- `accessibility-expert` - WCAG compliance, ARIA, screen readers
- `nextjs-expert` - Next.js App Router, Server Components, deployment


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [carlrannaberg/claudekit](https://github.com/carlrannaberg/claudekit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
