---
trigger: always_on
description: Quick reference for AI agents working in this repository.
---

# Agent Workflow Guide

Quick reference for AI agents working in this repository.

## Repository Structure

- `packages/` - Lerna monorepo packages
- `scripts/` - Build, test, and deployment utilities
- `codebuild_specs/` - CI/CD configuration

## Essential Commands

```sh
yarn build              # Build all packages
yarn test               # Run all tests
yarn lint-check         # Check linting
yarn setup-dev          # Setup local CLI (amplify-dev)
```

### Dependabot & Security Fixes

See [.agent-docs/DEPENDABOT.md](./.agent-docs/DEPENDABOT.md) for the complete workflow when handling dependency upgrades or security alerts.

Quick check:

```sh
npx ts-node scripts/check-dependabot.ts
```

## Standard Development Workflow

### 1. Research Stage

Start by delegating a context-gatherer sub agent to identify the relevant files to this issue. The sub agent must report both the files relevant to the issue, and the docs files relevant to the issue, as well as any additional context necessary to the issue.
When in doubt, ask the user clarifying questions. When you think you have enough context to implement the task, summarize it to the user and ask for confirmation before continuing.

**Before changing code you MUST first reference the relevant docs/ files**. Documentation is organized under `docs/` in the same path as the code it references. For example

- `packages/amplify-cli/src/commands/drift.ts`: `docs/packages/amplify-cli/src/commands/drift.md`

#### Finding Code

1. **Search symbols first:** Use `code` tool with `search_symbols` for functions/classes/types
2. **Follow with lookup:** Use `lookup_symbols` to get implementation details
3. **Grep for text:** Only for literal strings, comments, config values

##### Common Patterns

- CLI commands: `packages/amplify-cli/src/commands/`
- Category plugins: `packages/amplify-category-*/`
- Provider logic: `packages/amplify-provider-awscloudformation/`
- Test utilities: `packages/amplify-e2e-core/`, `packages/amplify-e2e-tests/`
- Scripts: `scripts/` (e2e-test-manager.ts, cloud-e2e.sh)

### 2. Implementation Stage

For non-trivial features, exhaust the design discussion before writing code. If the user says "don't implement yet" or "let's brainstorm," take that seriously — iterating on a design in conversation is far cheaper than iterating in code.
Only start implementing when the approach is settled and confirmed.

Make the necessary code changes and follow the guidelines in [CODING_GUIDELINES](./CODING_GUIDELINES.md).
For incremental validation, run `jest` commands directly and filter for the relevant tests.

### 3. Verification Stage

Verify your changes by following these guidelines:

- Run `yarn build && yarn test` in the package closest to the one you are working on.

### 4. Commit Stage

- **Always** update the appropriate JSDoc strings in the code you change. Be concise.
- Do not create additional markdown files in the repository unless you are instructed explicitly to.
- Never commit `.ai-generated` files (`.commit-message.ai-generated.txt`, `.pr-body.ai-generated.md`, etc.) — they are gitignored and are only used as local scratch files.
- Commit your changes in git using a well-formed commit message following the Conventional Commits format. The message must include
  a scope when the change is scoped to a specific package: `type(scope): subject`. The scope is derived from the package's `name`
  field in `package.json` with the `@aws-amplify/` prefix stripped. For example, `@aws-amplify/cli-internal` → `cli-internal`,
  `@aws-amplify/amplify-prompts` → `amplify-prompts`. Valid scopes are enforced by commitlint via `commitlint.config.js`. The
  message must start with a type prefix (e.g., `feat:`, `fix:`, `docs:`, `refactor:`, `test:`, `chore:`) followed by a single sentence summary and no more
  than a few paragraphs explaining the change and your testing. After this explanation, place the prompt the user used to trigger this
  work prefixed with a "Prompt: " after a single line consisting of '---'. Make sure there are no empty lines before or after this line.
  Word wrap all paragraphs at 72 columns including the prompt. For the author of the commit, use the configured username in git with
  ' (AI)' appended and the user email. For example, `git commit --author="John Doe (AI) <john@bigco.com>" -m "docs: update configuration guide"`.
  To avoid issues with multi-line commit messages, write the message to `.commit-message.ai-generated.txt` **at the repository root** and use `-F` with the path relative to your cwd:

  ```bash
  NODE_OPTIONS="--max-old-space-size=8192" git commit --author="John Doe (AI) <john@bigco.com>" -F <repo-root>/.commit-message.ai-generated.txt
  ```

  Always set `NODE_OPTIONS="--max-old-space-size=8192"` when committing to prevent OOM failures in the lint-staged hook.
  After a successful commit, delete the scratch file: `rm -f ../../.commit-message.ai-generated.txt` (adjust the relative path to point to the repo root).

- **CRITICAL: Always write `.commit-message.ai-generated.txt` to the repository root**, not inside a package directory. The `-F` path
  in `git commit -F` is resolved relative to the cwd, so adjust the relative path accordingly (e.g., `../../.commit-message.ai-generated.txt`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aws-amplify/amplify-cli](https://github.com/aws-amplify/amplify-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
