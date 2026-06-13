---
trigger: always_on
description: This repository contains an Obsidian plugin called Tasks Calendar that
---

# Development guide

This repository contains an Obsidian plugin called Tasks Calendar that
visualizes tasks on an interactive calendar using FullCalendar. It integrates
with the Dataview plugin to automatically gather tasks from notes across the
vault.

## Prerequisites & installation

See [README.md](./README.md#requirements) for prerequisites and installation
instructions.

## Build

```bash
npm run build         # Quick build with type checking (`skipLibCheck` enabled)
```

## Code quality checks

```bash
npm run check         # Comprehensive check: format + lint + strict type checking
npm run check:format  # Check formatting -- included in `npm run check`
npm run check:lint    # Check for ESLint errors -- included in `npm run check`
npm run check:tsc     # Strict type checking (no `skipLibCheck`) -- included in `npm run check`
```

## Code quality fixes

```bash
npm run fix           # Auto-format + auto-fix linting (combined)
npm run fix:format    # Auto-format code with Prettier -- included in `npm run fix`
npm run fix:lint      # Auto-fix ESLint errors -- included in `npm run fix`
```

## Testing

Currently, there are no agent-executable tests available, as testing typically
requires manual intervention. Instead, it is recommended to statically ensure
that the code works by following these steps:

1. Run `npm run build` to verify that the build completes without issues
2. Run `npm run check` to verify that there are no errors in the code
3. Run `npm run fix` to adjust code style

## Development

### General guidelines

- **Make sure to run the code quality checks after making changes**
  - During development: Run `npm run build` to verify compilation succeeds
  - Before finalizing/committing: Run `npm run check` for comprehensive
    validation. This runs format check, ESLint (0 warnings) and strict
    TypeScript type checking (no `skipLibCheck`)
  - To fix issues: Use `npm run fix` to auto-format and fix linting in one
    command
- Keep the plugin small. Avoid large dependencies. Prefer browser-compatible
  packages.
- Avoid Node/Electron APIs where possible.

### Coding style

**[!IMPORTANT]: ALWAYS REMEMBER WITH HIGH PRIORITY**

- All code, documentation and comments should be written in English
  - If instructions are given in a language other than English, you may respond
    in that language
  - But code/documentation/comments must be written in English unless explicitly
    requested in the instructions
- **Do not leave unnecessary comments in code**
  - Instead prefer self-documenting code with clear variable, function names,
    and data/control flows
- **When writing documentation, avoid excessive decoration**. For example, avoid
  scattering emojis or overusing `**` bold formatting. Use these only where
  truly necessary.
- **Use backticks for code references**: When writing comments, commit messages,
  or documentation, wrap code-related terms in backticks (e.g., `functionName`,
  `variableName`, `file.ts`) to distinguish them from regular text.
- **Commit messages**:
  - Do not include the "Generated with
    [Claude Code](https://claude.com/claude-code)" footer in commit messages for
    this project. Keep commit messages focused and concise.
  - When writing commit messages, follow the format `component: Brief summary`
    for the title. In the body of the commit message, provide a brief prose
    summary of the purpose of the changes made. Also, ensure that the maximum
    line length never exceeds 72 characters. When referencing external GitHub
    PRs or issues, use proper GitHub interlinking format (e.g., `owner/repo#123`
    for PRs/issues).
- Keep `main.ts` minimal: Focus only on plugin lifecycle (onload, onunload,
  addCommand calls). Delegate all feature logic to separate modules.
- Split large files: If any file exceeds ~200-300 lines, consider breaking it
  into smaller, focused modules.
- Use clear module boundaries: Each file should have a single, well-defined
  responsibility.
- Prefer `async/await` over promise chains; handle errors gracefully.
- **Minimize `try/catch` scope**: Only wrap operations that can actually throw
  errors. Extract the error-prone operation and use early return:

  ```ts
  // Good: minimal try/catch scope
  let result;
  try {
    result = await dangerousOperation();
  } catch (error) {
    logger.error(`Failed: ${error}`);
    return;
  }
  safeOperation(result);

  // Bad: unnecessarily wide try/catch
  try {
    const result = await dangerousOperation();
    safeOperation(result); // Should be outside try
  } catch (error) {
    logger.error(`Failed: ${error}`);
  }
  ```

- Generally, **efforts to maintain backward compatibility are not necessary
  unless explicitly requested by users**. For example, when renaming field names
  in data structures, you can simply perform the rename.

See also [Obsidian style guide](./obsidian-style-guide.md)

### Logging guidelines

All logging uses the `createLogger()` helper function from `src/logging.ts`,
which automatically adds `[TasksCalendar.ComponentName]` prefixes. Create a
logger instance in each class or component:

```typescript
import { createLogger } from './logging';

class MyComponent {
  private readonly logger = createLogger('MyComponent');

  someMethod() {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aviatesk/obsidian-tasks-calendar](https://github.com/aviatesk/obsidian-tasks-calendar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-13 -->
