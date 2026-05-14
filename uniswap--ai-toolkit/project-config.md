---
trigger: always_on
description: - **REQUIREMENT**: Use Nx for ALL packages and tooling in this monorepo
---

# CLAUDE.md - AI Toolkit Project Guidelines

## Core Requirements

### Nx Usage

- **REQUIREMENT**: Use Nx for ALL packages and tooling in this monorepo
- Every package must be an Nx project with proper configuration
- Use Nx generators, executors, and workspace features wherever possible
- Leverage Nx's dependency graph and caching capabilities

#### Nx Generator Schema Configuration - Critical Rules

When creating Nx generators with conditional prompting (showing/hiding prompts based on other values), follow these rules:

**CRITICAL: Avoid `x-prompt` for conditional fields**

The `x-prompt` property can interfere with conditional prompting logic. For fields that should only appear conditionally:

1. ❌ **DO NOT** use `x-prompt` (even nested as `x-prompt.when`) IF the variable is not ALWAYS something we want to know from the User (i.e. it's NOT CONDITIONAL)
2. ❌ **DO NOT** include a `default` value if the field should be optional
3. ✅ **DO** use only `prompt-when` at the property level
4. ✅ **DO** rely on the prompt-utils to generate prompts from property metadata

**Example - Conditional Prompt Implementation:**

```json
{
  "properties": {
    "installMode": {
      "type": "string",
      "enum": ["all", "specific"],
      "default": "all",
      "x-prompt": {
        "message": "What would you like to install?",
        "type": "list",
        "items": [...]
      }
    },
    "addon": {
      "type": "string",
      "enum": ["option1", "option2", "option3"],
      // ❌ NO x-prompt here - it would always trigger
      // ❌ NO default value - would prevent prompt from showing when needed
      "prompt-when": "installMode === 'specific'"  // ✅ ONLY this
    }
  }
}
```

**Why this matters:**

- `x-prompt` presence can cause prompts to appear regardless of conditions
- `default` values can interfere with the `shouldPrompt` logic in prompt-utils
- The prompt-utils checks for `prompt-when` AFTER checking if prompting is needed
- Only properties without `x-prompt` and without defaults will properly respect `prompt-when`

**Reference:** See packages/ai-toolkit-nx-claude/src/generators/addons/schema.json for a working example of conditional prompting with `installMode` controlling `addon` visibility.

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

1. **Format the code**: Run `bunx nx format:write --uncommitted` to format all uncommitted files using Prettier configuration

   - For CI or when comparing against a specific base: `bunx nx format:write --affected --base=HEAD~1`
   - This ensures consistent code style across the entire codebase

2. **Lint the code**: Run `bunx nx affected --target=lint --base=HEAD~1` to check for linting errors

   - This runs ESLint on all affected projects based on your changes
   - If there are linting errors that can be auto-fixed, add the `--fix` flag: `bunx nx affected --target=lint --base=HEAD~1 --fix`

3. **Typecheck the code**: Run `bunx nx affected --target=typecheck --base=HEAD~1` to typecheck all affected projects.

4. **Lint markdown files**: After creating or updating any markdown files (.md), run `bunx markdownlint-cli2 --fix "**/*.md"` to ensure all markdown files follow the project's markdown standards.

   - This uses the `.markdownlint-cli2.jsonc` configuration file at the repository root
   - The linting will automatically fix most common formatting issues
   - This check MUST pass (exit code 0) before proceeding
   - The markdown linter also runs automatically as a pre-commit hook via Lefthook

5. **Why use these commands**:
   - `--uncommitted` flag formats only files that haven't been committed yet
   - `--affected` with `--base=HEAD~1` identifies all projects affected by changes since the last commit
   - Using Nx's affected commands ensures only relevant code is checked, making the process fast and efficient
   - These commands leverage Nx's dependency graph to include downstream projects that might be affected

## Project Context

This is the AI Toolkit monorepo that provides standardized Claude Code plugins via the Claude Code Marketplace. The project uses:

- Nx for monorepo management
- TypeScript with strict settings
- Bun as the package manager (npm retained only for OIDC trusted publishing in CI)
- ESLint and Prettier for code quality

### Bun Package Manager

**This project uses Bun as its package manager.**

**Installation:**

```bash
# Install Bun (if not already installed)
curl -fsSL https://bun.sh/install | bash
```

**Key commands:**

```bash
bun install              # Install dependencies
bun install --frozen-lockfile  # CI-safe install (fails if bun.lock is out of date)
bunx nx <command>        # Run Nx commands
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Uniswap/ai-toolkit](https://github.com/Uniswap/ai-toolkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
