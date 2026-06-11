---
trigger: always_on
description: - Be extremely concise. Sacrifice grammar for the sake of concision.
---

# AGENTS.md - Developer Guidelines

- Be extremely concise. Sacrifice grammar for the sake of concision.

## Build/Test Commands

- `bun lint` - Lint with oxlint (run after changes) (required)
- `bun typecheck` - TypeScript type checking (required)
- `bun format` - Auto-format code with dprint
- `bun test` - Run all tests
- Single app commands: `bun --filter="@apps/bot" run lint` or `bun --filter="@apps/dashboard" run typecheck`

## Workflow

- **MUST**: Do not start any dev servers unless told otherwise.
- **MUST**: typecheck when you’re done making a series of code changes.
- **MUST**: Default to using Bun instead of Node.js.
- **MUST**: If you get stuck on literally anything, **YOU MUST** stop what you're doing and ask for a response from me. **Do not** resort to writing mock code!!
- **MUST**: Use no emojis in console output, and use `consola` for logging.
- **MUST**: No `any` or `unknown` types allowed.
- **MUST**: You must import types from `oceanic.js` for typings.
- **MUST**: Make sure to make migrations for prisma on schema changes.
- Prefer running single tests, and not the whole test suite, for performance
- Don't waste time on git actions like git status, git add, git commit, etc. Unless told otherwise.

## Code Style

- **Files**: kebab-case for .ts files, PascalCase for .tsx/.jsx
- **Formatting**: 2 spaces, single quotes, no trailing commas, semicolons as needed
- **Imports**: Use `#framework` for internal bot framework imports and type imports, like `Context`, `defineSlashCommand`, `Client`.
- **Types**: Use `type` imports (`import type`), no non-null assertions, explicit enum initializers, no `any` unless absolutely necessary.
- Small one sentence comments for complex logic that isn't understandable at first glance
- Comments should have no punctuation, no periods, no capita letters. No Oxford English boxing the sentences, just clear, coherent sentences that are short and easy to understand.
- Single quotes, no semicolons for TypeScript code. Vue 3 blocks should be in `<script setup lang="ts">` -> `<template>` -> `<style scoped>`.
- Use arrow functions for methods and computed properties.
- Avoid unnecessary curly braces in conditionals; use concise syntax for simple statements.
- Write concise, technical TypeScript code with accurate examples.
- Prefer iteration and modularization over code duplication.
- Use descriptive variable names with auxiliary verbs (e.g., isLoading, hasError).
- Use ES modules (import/export) syntax, not CommonJS (require)
- Destructure imports when possible (eg. import { foo } from 'bar')
- Try to avoid `any`, `unknown`, and `never` types
- Use `import type` instead of `import` for types.
- Don't dynamically import modules, use `import` instead at the top of the file.

Design rules for Discord bot responses:

1. Conciseness
   - Keep responses short and scannable.
   - Do not send walls of text unless explicitly requested.

2. Tone
   - Use casual and friendly language.
   - Grammar should be simple and clear.
   - Avoid overly formal or corporate tone.

3. Emojis
   - Use only when they add clarity or hierarchy (e.g., ✅ ❌ 🔔).
   - Do not spam or force emojis into every response.

4. Formatting
   - Bold (**text**) for emphasis.
   - Italics (_text_) for subtle hints or disclaimers.
   - Inline code (`command`) and code blocks for commands or outputs.
   - Blockquotes (> text) for context or references.

5. Subtext / Footnotes
   - Use “-# text” for secondary notes or disclaimers.
   - Keep the main response clean and focused.

6. Errors
   - Provide short, clear error messages.
   - Application-specific errors should not be shown to users.
   - Suggest corrective action (e.g., “Couldn’t find that user. Try @mention instead.”).

7. Feedback
   - Confirm user actions with short success messages.
   - Do not leave silent confirmations.

8. Response Rate
   - Throttle repetitive or spammy outputs.
   - Combine related information into one message instead of many.

9. Embeds
   - Use embeds for structured or complex data.
   - Avoid using embeds for casual chat.
   - Do not overuse embeds as it feels robotic.

10. Grammar
    - Use simple, declarative sentences.
    - Avoid semicolons and overly complex clauses.

11. Personality
    - Light personality is acceptable.
    - Do not sacrifice clarity for quirkiness.

## Architecture Patterns

- **Commands**: Inherit from SlashCommand, use `defineSlashCommand()` wrapper
- **Context**: Use ctx parameter for interactions, reply with `ctx.reply()`
- **Modules**: Access via `ctx.client.modules.{module}` (economy, analytics, etc.)
- **Database**: Prisma models, use workspace packages `@packages/database`
- **Error handling**: Use proper TypeScript types, avoid `any`, handle async/await correctly

## Path Aliases

- `#framework` → `./src/framework` (This is NOT a module alias import, its an entrypoint alias)

### For bun

- Use `bun <file>` instead of `node <file>` or `ts-node <file>`
- Use `bun test` instead of `jest` or `vitest`
- Use `bun build <file.html|file.ts|file.css>` instead of `webpack` or `esbuild`
- Use `bun install` instead of `npm install` or `yarn install` or `bun install`
- Use `bun run <script>` instead of `npm run <script>` or `yarn run <script>` or
  `bun run <script>`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [taskylizard/kanikou](https://github.com/taskylizard/kanikou) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-10 -->
