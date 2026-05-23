---
trigger: always_on
description: VS Code Extension for Azure Cosmos DB and the MongoDB API. TypeScript (strict mode), React webviews, Jest testing.
---

# GitHub Copilot Instructions for vscode-documentdb

VS Code Extension for Azure Cosmos DB and the MongoDB API. TypeScript (strict mode), React webviews, Jest testing.

## Critical Build Commands

| Command                | Purpose                                                      |
| ---------------------- | ------------------------------------------------------------ |
| `npm run build`        | **Build the project** (use this, NOT `npm run compile`)      |
| `npm run lint`         | Check for linting errors                                     |
| `npm run prettier-fix` | Format code                                                  |
| `npm run l10n`         | Update localization files after changing user-facing strings |

> ⚠️ **NEVER use `npm run compile`** - always use `npm run build` to build the project.

## PR Completion Checklist

Before finishing work on a PR, agents **must** run the following steps in order:

1. **Localization** — If any user-facing strings were added, modified, or removed, run:
   ```bash
   npm run l10n
   ```
2. **Formatting** — Run Prettier to ensure all files meet formatting standards:
   ```bash
   npm run prettier-fix
   ```
3. **Linting** — Run ESLint to confirm there are no linting errors:
   ```bash
   npm run lint
   ```
4. **Tests** — Run the full Jest test suite to confirm no regressions:
   ```bash
   npx jest --no-coverage
   ```
5. **Build** — Run the TypeScript build to confirm there are no type errors:
   ```bash
   npm run build
   ```

> ⚠️ **An agent must not finish or terminate until all five steps above have been run and pass successfully.** Skipping these steps leads to CI failures.

## Git Safety

- **Never use `git add -f`** to force-add files. If `git add` refuses a file, it is likely in `.gitignore` for a reason (e.g., `docs/plan/`, `docs/analysis/`, build outputs). Do NOT override this with `-f`.
- When `git add` warns that a path is ignored, **stop and inform the user** instead of force-adding.
- Files in `docs/plan/` and `docs/analysis/` are **local planning documents** that must not be committed to the repository.

## Project Structure

| Folder          | Purpose                                    |
| --------------- | ------------------------------------------ |
| `src/`          | Main extension source code                 |
| `src/webviews/` | React web view components                  |
| `src/commands/` | Command handlers (one folder per command)  |
| `src/services/` | Singleton services                         |
| `src/tree/`     | Tree view data providers                   |
| `api/`          | Separate Node.js project for extension API |
| `l10n/`         | Localization files                         |
| `test/`         | Jest tests                                 |

## Branching

- **`main`**: Default branch; all PRs target it.
- **`release/<X.Y>`**: Short-lived stabilization branches cut from `main` at freeze; release tags (`vX.Y.0`, `vX.Y.1`, …) live here.

## TypeScript Guidelines

- **Never use `any`** - use `unknown` with type guards
- **Prefer `interface`** for object shapes, `type` for unions
- **Always specify return types** for functions
- **Use `vscode.l10n.t()`** for all user-facing strings

```typescript
// ✅ Good - Interface with explicit types
interface ConnectionConfig {
  readonly host: string;
  readonly port: number;
}

// ✅ Good - Named function with return type
export function createConnection(config: ConnectionConfig): Promise<Connection> {
  // implementation
}

// ✅ Good - Localized user-facing string with safe error handling
try {
  await operation();
} catch (error) {
  const errorMessage = error instanceof Error ? error.message : String(error);
  void vscode.window.showErrorMessage(vscode.l10n.t('Failed to connect: {0}', errorMessage));
}
```

## Null Safety

Use `nonNullProp()`, `nonNullValue()`, `nonNullOrEmptyValue()` from `src/utils/nonNull.ts`:

```typescript
// ✅ Good - Use nonNull helpers for internal validation
const connectionString = nonNullProp(
  selectedItem.cluster,
  'connectionString',
  'selectedItem.cluster.connectionString',
  'ExecuteStep.ts',
);

// ✅ Good - Manual check for user-facing validation with l10n
if (!userInput.connectionString) {
  void vscode.window.showErrorMessage(vscode.l10n.t('Connection string is required'));
  return;
}
```

## Error Handling

When accessing error properties in catch blocks or error handlers, always check if the error is an instance of `Error` before accessing `.message`:

```typescript
// ✅ Good - Type-safe error message extraction
try {
  await someOperation();
} catch (error) {
  const errorMessage = error instanceof Error ? error.message : String(error);
  void vscode.window.showErrorMessage(vscode.l10n.t('Operation failed: {0}', errorMessage));
}

// ✅ Good - In promise catch handlers
void task.start().catch((error) => {
  const errorMessage = error instanceof Error ? error.message : String(error);
  void vscode.window.showErrorMessage(vscode.l10n.t('Failed to start: {0}', errorMessage));
});

// ❌ Bad - Direct access to error.message (eslint error)
catch (error) {
  void vscode.window.showErrorMessage(vscode.l10n.t('Failed: {0}', error.message)); // Unsafe!
}
```

## Command Pattern


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/vscode-documentdb](https://github.com/microsoft/vscode-documentdb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
