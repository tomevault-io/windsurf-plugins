---
trigger: always_on
description: - Target: Obsidian Community Plugin (TypeScript → bundled JavaScript).
---

## Project Documentation

## Project overview

- Target: Obsidian Community Plugin (TypeScript → bundled JavaScript).
- Entry point: `main.ts` compiled to `main.js` and loaded by Obsidian.
- Required release artifacts: `main.js`, `manifest.json`, and optional `styles.css`.

## Agent Workflow

### Files to ignore

\*\*NEVER read or modify

- `TODO.md`\*\* at the project root.
- Files under `documentation/archived` unless instructed otherwise.

### Documentation

**IMPORTANT**:

- At the start of each new session, read the history files in `documentation/history` and plans in `documentation/plans` to understand the project, its current state and what should come next
- Whenever making plans, DO NOT include timing information (e.g., 1-3 weeks, etc) UNLESS asked explicitly
- Whenever making plans, store or update those in `documentation/plans`
- Whenever making plans, focus on actionable information

History is maintained in `documentation/history/yyyy-mm-dd.md` files, organized chronologically. Each file documents:

- What was accomplished that day
- Key decisions made
- Domain model changes
- Implementation progress
- Open questions or blockers

These files are optimized for conciseness and clarity to quickly onboard agents in new sessions.

### Business Rules Compliance

**CRITICAL**: Business rules documented in `documentation/Business Rules.md` MUST ALWAYS be respected unless explicit user approval is given to change or bypass them.

- **Mandatory compliance**: All implementations must respect documented business rules
- **No exceptions without approval**: Changing or bypassing any business rule requires explicit user approval
- **Highest priority**: When making changes, business rule compliance is of the utmost importance
- **Documentation requirement**: When a new business rule is mentioned, it must be immediately documented in `documentation/Business Rules.md` using a concise format (single line or paragraph) without losing precision

Read the Business Rules document at the start of each session to understand the constraints and requirements.

**MUST READ** before working on this codebase: `documentation/**/*.md` — system overview, architecture, components, directory structure, configuration, settings, ...

**MUST UPDATE** documentation when making changes. Keep it terse, accurate, no fluff.

### Core Coding Rules

**MUST READ** TypeScript type definitions you can find in the Obsidian dependency now.

## Environment & tooling

- **[Bun](https://bun.com/)**: a fast all-in-one JavaScript runtime.
- **Package manager: Bun** (required for this sample - `package.json` defines scripts and dependencies).
- **Bundler: Bun** (required for this sample - `build.ts` depends on it).
- **Types**: `obsidian` type definitions.

### Install

```bash
bun install
```

### Dev (watch)

```bash
bun run dev
```

### Production build

```bash
bun run build
```

## Development Workflow

**CRITICAL**: Before making ANY code changes, start the TypeScript watch process in the background:

```bash
bun run tsc:watch
```

This is MANDATORY. The watch process catches type errors immediately as you edit. Check the output after each edit to catch errors early. If you see TypeScript errors, fix them before moving on.

Optionally, also run tests in watch mode:

```bash
bun test --watch
```

After editing code, always run the formatter and linter:

```bash
bun run format
bun run lint
```

Both commands are **MANDATORY** after code changes. Fix any lint errors before proceeding.

## Bun Runtime

Default to using Bun instead of Node.js.

- Use `bun <file>` instead of `node <file>` or `ts-node <file>`
- Use `bun test` instead of `jest` or `vitest`
- Use `bun build <file.html|file.ts|file.css>` instead of `webpack` or `esbuild`
- Use `bun install` instead of `npm install` or `yarn install` or `pnpm install`
- Use `bun run <script>` instead of `npm run <script>` or `yarn run <script>` or `pnpm run <script>`
- Use `bunx <package> <command>` instead of `npx <package> <command>`
- Bun automatically loads .env, so don't use dotenv.

## Testing

Use `bun test` to run tests.

**MANDATORY**: All test files MUST use the `.spec.ts` extension (not `.test.ts`).

```ts#example.spec.ts
import { test, expect } from "bun:test";

test("hello world", () => {
  expect(1).toBe(1);
});
```

Test files should be placed next to the files they test:

```
scripts/
  build.ts
  build.spec.ts        # Tests for build.ts
  update-version.ts
  update-version.spec.ts
```

- Manual install for testing: copy `main.js`, `manifest.json`, `styles.css` (if any) to:
    ```
    <Vault>/.obsidian/plugins/<plugin-id>/
    ```
- Reload Obsidian and enable the plugin in **Settings → Community plugins**.

## File & folder conventions

### Base organization

- **Organize code into multiple files**: Split functionality across separate modules rather than putting everything in `main.ts`.
- Source lives in `src/`. Keep `main.ts` small and focused on plugin lifecycle (loading, unloading, registering commands).
- All CSS lives in `src/styles.src.css` (Tailwind source file)
- **CSS/Styles**: ALWAYS edit styles in `src/styles.src.css` (Tailwind source file), NEVER edit `styles.css` at the root (this is a generated file that gets overwritten during build).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dsebastien/obsidian-hidden-folders-access](https://github.com/dsebastien/obsidian-hidden-folders-access) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
