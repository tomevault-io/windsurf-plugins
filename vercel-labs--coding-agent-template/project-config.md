---
trigger: always_on
description: This document contains critical rules and guidelines for AI agents working on this codebase.
---

# AI Agent Guidelines

This document contains critical rules and guidelines for AI agents working on this codebase.

## Security Rules

### CRITICAL: No Dynamic Values in Logs

**All log statements MUST use static strings only. NEVER include dynamic values, regardless of severity.**

#### Bad Examples (DO NOT DO THIS):
```typescript
// BAD - Contains dynamic values
await logger.info(`Task created: ${taskId}`)
await logger.error(`Failed to process ${filename}`)
console.log(`User ${userId} logged in`)
console.error(`Error for ${provider}:`, error)
```

#### Good Examples (DO THIS):
```typescript
// GOOD - Static strings only
await logger.info('Task created')
await logger.error('Failed to process file')
console.log('User logged in')
console.error('Error occurred:', error)
```

#### Rationale:
- **Prevents data leakage**: Dynamic values in logs can expose sensitive information (user IDs, file paths, credentials, etc.) to end users
- **Security by default**: Logs are displayed directly in the UI and returned in API responses
- **No exceptions**: This applies to ALL log levels (info, error, success, command, console.log, console.error, console.warn, etc.)

#### Sensitive Data That Must NEVER Appear in Logs:
- Vercel credentials (SANDBOX_VERCEL_TOKEN, SANDBOX_VERCEL_TEAM_ID, SANDBOX_VERCEL_PROJECT_ID)
- User IDs and personal information
- File paths and repository URLs
- Branch names and commit messages
- Error details that may contain sensitive context
- Any dynamic values that could reveal system internals

### Credential Redaction

The `redactSensitiveInfo()` function in `lib/utils/logging.ts` automatically redacts known sensitive patterns, but this is a **backup measure only**. The primary defense is to never log dynamic values in the first place.

#### Current Redaction Patterns:
- API keys (ANTHROPIC_API_KEY, OPENAI_API_KEY, etc.)
- GitHub tokens (ghp_, gho_, ghu_, ghs_, ghr_)
- Vercel credentials (SANDBOX_VERCEL_TOKEN, SANDBOX_VERCEL_TEAM_ID, SANDBOX_VERCEL_PROJECT_ID)
- Bearer tokens
- JSON fields (teamId, projectId)
- Environment variables containing KEY, TOKEN, SECRET, PASSWORD, TEAM_ID, PROJECT_ID

## Code Quality Guidelines

### Code Formatting and Quality Checks

**Always run `pnpm format`, `pnpm type-check`, and `pnpm lint` after making changes to TypeScript/TSX files.**

The project uses Prettier for code formatting, TypeScript for type checking, and ESLint for linting. After editing any `.ts` or `.tsx` files, run:

```bash
pnpm format
pnpm type-check
pnpm lint
```

**If any errors are found:**
1. **Type errors**: Fix TypeScript type errors by correcting type annotations, adding missing imports, or fixing type mismatches
2. **Lint errors**: Fix ESLint errors by following the suggested fixes or adjusting the code to meet the linting rules
3. **Do not skip or ignore errors** - all errors must be resolved before considering the task complete

This ensures all code follows the project's formatting standards, type safety requirements, and linting rules, preventing issues in pull requests.

### Use shadcn CLI for UI Components

**When adding UI components, check if a shadcn/ui component exists and install it via CLI instead of writing it manually.**

```bash
pnpm dlx shadcn@latest add <component-name>
```

Existing components are in `components/ui/`. See [shadcn/ui docs](https://ui.shadcn.com/) for available components.

### CRITICAL: Never Run Dev Servers

**DO NOT run development servers (e.g., `npm run dev`, `pnpm dev`, `next dev`) as they will conflict with other running instances.**

#### Why This Rule Exists:
- Dev servers run indefinitely and block the terminal session
- Multiple instances on the same port cause conflicts
- The application may already be running in the user's environment
- Long-running processes make the conversation hang for the user

#### Commands to AVOID:
```bash
# DO NOT RUN THESE:
npm run dev
pnpm dev
next dev
npm start
pnpm start
yarn dev
node --watch
nodemon
```

#### What to Do Instead:
1. **Testing changes**: Use `pnpm build` to verify the production build works
2. **Type checking**: Use `pnpm type-check` to verify types
3. **Linting**: Use `pnpm lint` to check code quality
4. **Running tests**: Use `pnpm test` if tests are available
5. **If the user needs to test**: Let the user run the dev server themselves

#### Exception:
If the user explicitly asks you to start a dev server, politely explain why you cannot do this and suggest they run it themselves instead.

### Logging Best Practices

1. **Use descriptive static messages**
   ```typescript
   // Instead of logging the value, log the action
   await logger.info('Sandbox created successfully')
   await logger.info('Dependencies installed')
   await logger.error('Build failed')
   ```

2. **Server-side logging for debugging**
   ```typescript
   // Use console.error for server-side debugging (not shown to users)
   // But still avoid sensitive data
   console.error('Sandbox creation error:', error)
   ```

3. **Progress updates**
   ```typescript
   // Use static progress messages
   await logger.updateProgress(50, 'Installing dependencies')
   await logger.updateProgress(75, 'Running build')
   ```

### Error Handling

1. **Generic error messages to users**
   ```typescript

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vercel-labs/coding-agent-template](https://github.com/vercel-labs/coding-agent-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
