---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is `eslint-plugin-harlanzw` - Harlan's experimental ESLint rules for Vue projects, particularly focused on link hygiene, Nuxt best practices, and Vue reactivity patterns. The plugin provides 23 link/nuxt/vue rules, 12 AI deslop rules for cleaning AI-generated content, and 21 prompt linting rules.

## Development Commands

```bash
# Install dependencies
pnpm install

# Build the plugin
pnpm build

# Development mode (stub build for faster iteration)
pnpm dev

# Run tests
pnpm test

# Lint the codebase (requires pnpm dev first)
pnpm lint

# Type checking
pnpm typecheck

# Release process (build + version bump + publish)
pnpm release
```

## Testing

- Uses **Vitest** as the test runner with globals enabled
- Uses **eslint-vitest-rule-tester** for testing ESLint rules
- Test files are located alongside rule files with `.test.ts` suffix
- Common test utilities are in `src/rules/_test.ts`
- Each rule should have comprehensive test cases covering valid/invalid scenarios
- Every new rule **must** have a `.test.ts` file and an entry in the rules table in `README.md`

## Project Architecture

### Core Structure
- `src/index.ts` - Main plugin export, registers all rules
- `src/rules/` - Individual ESLint rule implementations
- `src/utils.ts` - Rule creation utilities and helpers
- `src/vue-utils.ts` - Vue-specific AST utilities and helpers
- `src/ast-utils.ts` - General AST manipulation utilities

### Rule Development Pattern
Each rule follows a consistent pattern:
1. Rule file (e.g., `vue-no-ref-access.ts`)
2. Test file (e.g., `vue-no-ref-access.test.ts`)
3. Documentation file (e.g., `vue-no-ref-access.md`) - for rules with extensive docs
4. Rules are created using the `createEslintRule` helper from `src/utils.ts`

### Plugin Rules (23 link/nuxt/vue + 10 deslop + 21 prompt)

**Link rules (8):**
- `link-ascii-only` - Ensure link URLs contain only ASCII characters
- `link-lowercase` - Ensure link URLs do not contain uppercase characters
- `link-no-double-slashes` - Ensure link URLs do not contain consecutive slashes
- `link-no-underscores` - Ensure link URLs do not contain underscores
- `link-no-whitespace` - Ensure link URLs do not contain whitespace characters
- `link-require-descriptive-text` - Require descriptive link text
- `link-require-href` - Require href/to attribute on link elements
- `link-trailing-slash` - Enforce trailing slash consistency on link URLs

**Nuxt rules (8):**
- `nuxt-await-navigate-to` - Enforces awaiting navigateTo() calls in Nuxt
- `nuxt-no-random` - Prevents Math.random() in SSR contexts (hydration mismatch)
- `nuxt-no-redundant-import-meta` - Prevents redundant import.meta checks in scoped components
- `nuxt-no-side-effects-in-async-data-handler` - Prevents side effects in async data handlers
- `nuxt-no-side-effects-in-setup` - Prevents side effects in Vue setup functions
- `nuxt-no-unsafe-date` - Warns against Date APIs that differ between server/client
- `nuxt-prefer-navigate-to-over-router-push-replace` - Prefer navigateTo over router methods
- `nuxt-prefer-nuxt-link-over-router-link` - Prefer NuxtLink over RouterLink components

**Vue rules (7):**
- `vue-no-faux-composables` - Prevents fake composables that don't use Vue reactivity
- `vue-no-nested-reactivity` - Prevents mixing ref() and reactive() patterns
- `vue-no-passing-refs-as-props` - Requires unwrapping refs before passing as props
- `vue-no-reactive-destructuring` - Prevents destructuring reactive objects (loses reactivity)
- `vue-no-ref-access-in-templates` - Prevents using .value in Vue templates
- `vue-no-torefs-on-props` - Prevents using toRefs() on props object
- `vue-require-composable-prefix` - Enforces use* prefix for functions using Vue reactivity

**AI Deslop rules (10):** Target `content/**/*.md` — clean AI-generated slop from content
- `ai-deslop-adverbs` - Remove unnecessary adverbs ("significantly", "fundamentally")
- `ai-deslop-autolink` - Auto-link first mention of tech terms to canonical URLs
- `ai-deslop-buzzwords` - Replace AI buzzwords with plain alternatives ("leverage" → "use")
- `ai-deslop-casing` - Enforce correct tech term casing ("github" → "GitHub")
- `ai-deslop-filler` - Remove filler phrases ("it's worth noting that", "at the end of the day")
- `ai-deslop-hedging` - Remove hedging/qualifying words ("very", "really", "quite", "just")
- `ai-deslop-no-em-dash` - Replace em dashes with plain dashes ("—" → " - ")
- `ai-deslop-no-exclamation` - Remove exclamation marks from content prose
- `ai-deslop-passive-voice` - Flag passive voice constructions ("is generated" → rewrite actively)
- `ai-deslop-weak-opener` - Flag weak sentence openers ("There is", "It is possible to")
- `ai-deslop-frontmatter-spacing` - Remove empty lines inside YAML frontmatter
- `ai-deslop-vue-ts-lang` - Require `lang="ts"` on Vue `<script>` blocks in code examples

**Shared configs:** `link`, `nuxt`, `vue`, `recommended` (all three), `content` (deslop), `prompt:recommended`, `prompt:strict`, `prompt:skill`

### Key Utilities
- `VUE_REACTIVITY_APIS` - Set of all Vue reactivity function names

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [harlan-zw/eslint-plugin-harlanzw](https://github.com/harlan-zw/eslint-plugin-harlanzw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
