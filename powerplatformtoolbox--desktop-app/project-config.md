---
trigger: always_on
description: **Power Platform ToolBox** is an Electron-based desktop application (v28) that provides a universal platform for Power Platform development tools. It features a VS Code Extension Host-inspired architecture for secure, isolated tool execution. The app is built with TypeScript targeting ES2022 and requires Node.js 18+.
---

# Power Platform ToolBox - Copilot Instructions

## Repository Overview

**Power Platform ToolBox** is an Electron-based desktop application (v28) that provides a universal platform for Power Platform development tools. It features a VS Code Extension Host-inspired architecture for secure, isolated tool execution. The app is built with TypeScript targeting ES2022 and requires Node.js 18+.

**Size**: ~10,000+ lines of code across TypeScript source files
**Primary Language**: TypeScript (strict mode enabled)
**Runtime**: Electron 28, Node.js 18+
**Key Technologies**: Electron, TypeScript, electron-store (settings), electron-updater (auto-updates), @azure/msal-node (authentication), Vite (build tool), Sass (styling)

## CRITICAL: Code Quality Standards

### Logging and Error Handling

**NEVER use console.log, console.warn, or console.error in production code.** Instead, use appropriate Sentry methods for telemetry and error tracking:

- **For informational logs**: Use `Sentry.captureMessage(message, 'info')` or appropriate logging mechanism
- **For warnings**: Use `Sentry.captureMessage(message, 'warning')`
- **For errors**: Use `Sentry.captureException(error)` to capture full error context with stack traces
- **For debug information**: Only use console methods during development with clear comments indicating they should be removed

**Example - BAD:**

```typescript
console.log("User settings loaded");
console.warn("Connection token expired");
console.error("Failed to load tool:", error);
```

**Example - GOOD:**

```typescript
Sentry.captureMessage("User settings loaded successfully", "info");
Sentry.captureMessage("Connection token expired for user", "warning");
Sentry.captureException(new Error("Failed to load tool"), {
    extra: { toolId, error: error.message },
});
```

### ESLint and Prettier Configuration

**ALWAYS format code according to project standards before committing:**

- **Prettier Configuration** (`.prettierrc.json`):
    - **printWidth**: 200 characters (long lines are OK)
    - **tabWidth**: 4 spaces (NOT 2)
    - **singleQuote**: false (use double quotes)
    - **semi**: true (always use semicolons)
    - **trailingComma**: "all" (add trailing commas)
    - **endOfLine**: "auto" (cross-platform compatibility)

- **ESLint Configuration** (`.eslintrc.js`):
    - Parser: `@typescript-eslint/parser`
    - Strict TypeScript rules enabled
    - **`@typescript-eslint/no-explicit-any`**: "warn" (warnings OK, but avoid when possible)
    - Target: ES2020, Node.js environment

**Before any commit or code generation:**

1. Run `pnpm run lint` - Must complete with 0 errors (warnings are acceptable)
2. Format code with Prettier (4-space tabs, double quotes, semicolons, trailing commas)
3. Verify TypeScript strict mode compliance

**Example Formatting:**

```typescript
// CORRECT ✓
export interface UserSettings {
    theme: string;
    language: string;
    autoUpdate: boolean;
}

async function loadUserData(): Promise<UserSettings> {
    const settings = await settingsManager.getUserSettings();
    return settings;
}

// INCORRECT ✗ (2-space tabs, single quotes, missing trailing comma)
export interface UserSettings {
    theme: string;
    language: string;
    autoUpdate: boolean;
}

async function loadUserData(): Promise<UserSettings> {
    const settings = await settingsManager.getUserSettings();
    return settings;
}
```

## Build System & Commands

### Prerequisites

- Node.js 18 or higher (currently tested with v20.19.5)
- pnpm 10.18.3 or higher (package manager - REQUIRED)
- **Supabase credentials** (required for tool registry):
    - `SUPABASE_URL` - Your Supabase project URL
    - `SUPABASE_ANON_KEY` - Your Supabase anonymous key

### Environment Variables

Create a `.env` file in the project root (gitignored) with:

```bash
SUPABASE_URL=https://your-project.supabase.co
SUPABASE_ANON_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

**Security Note**: These values are injected at build time via Vite and NOT stored in source code. For GitHub Actions, configure them as repository secrets.

### Installation & Build Sequence

**ALWAYS run these commands in this exact order for a clean build:**

```bash
pnpm install         # Install dependencies (~40s with pnpm)
pnpm run typecheck   # TypeScript type checking (main + renderer)
pnpm run lint        # Lint code (must have 0 errors, warnings OK)
pnpm run build       # Build the application using Vite (~2-5s)
```

### Available Commands

- **`pnpm install`** - Install all dependencies. Takes ~40s on first install. ALWAYS run after cloning or pulling package changes.
- **`pnpm run typecheck`** - Run TypeScript compiler in check mode for both main and renderer processes. No output files generated.
- **`pnpm run build`** - Complete production build. Runs typecheck + Vite build for main, preload, and renderer processes. Takes 2-5 seconds.
- **`pnpm run build:debug`** - Development build with source maps enabled for debugging.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PowerPlatformToolBox/desktop-app](https://github.com/PowerPlatformToolBox/desktop-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
