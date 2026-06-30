---
trigger: always_on
description: > This file provides guidance to GitHub Copilot when working with this repository.
---

# Copilot Instructions

> This file provides guidance to GitHub Copilot when working with this repository.

## Language Preference

**IMPORTANT: All responses, PR descriptions, commit message bodies, and comments MUST be written in Chinese (Simplified).**

> Note: Commit message type and scope should remain in English (e.g., `feat(proxy):`, `fix(auth):`), only the subject and body can be in Chinese.

---

## Request Handling

When mentioned with `@copilot` in PR comments, follow this unified workflow:

### Workflow: Review → Fix → Summarize

**Trigger keywords**: `审核`, `review`, `检查`, `看一下`, `帮我看`, `修复`, `fix`, `改一下`, `修改`, `实现`, `添加`

**Actions**:

1. **Review** - Analyze the code changes and identify issues
2. **Fix** - Create fixes for identified issues (if any)
3. **Summarize** - Reply with a summary in Chinese, including:
   - Issues found
   - Changes made
   - Recommendations

### Example Requests

| User says                 | Action                   |
| ------------------------- | ------------------------ |
| `@copilot 审核这个PR`     | Review → Fix → Summarize |
| `@copilot review this PR` | Review → Fix → Summarize |
| `@copilot 帮我看看代码`   | Review → Fix → Summarize |
| `@copilot 修复这个问题`   | Review → Fix → Summarize |

---

## Commit Message Convention

This repository uses [Conventional Commits](https://www.conventionalcommits.org/) specification.

**All commits MUST follow this format:**

```
<type>(<scope>): <subject>

[optional body]

[optional footer]
```

### Rules

1. **Use proper commit format** (Note: Copilot's built-in "Initial plan" commit is an internal Coding Agent mechanism and is automatically ignored by commitlint)

   Avoid these formats for manual commits:
   - `WIP`
   - `temp`
   - `test`
   - Any message without a type prefix

2. **Use one of these types:**
   - `feat:` - New feature (triggers minor version)
   - `fix:` - Bug fix (triggers patch version)
   - `docs:` - Documentation only (no release)
   - `style:` - Code style (no release)
   - `refactor:` - Code refactoring (triggers patch version)
   - `perf:` - Performance improvement (triggers patch version)
   - `test:` - Adding tests (no release)
   - `chore:` - Maintenance tasks (no release)
   - `ci:` - CI configuration (no release)

3. **Subject rules:**
   - Use imperative mood ("add" not "added")
   - Don't capitalize first letter
   - No period at the end
   - 5-100 characters

### Examples

✅ Good:

```
feat(proxy): add cookie sticky session support
fix(auth): handle expired token refresh
refactor(api): simplify error handling
chore: update dependencies
```

❌ Bad (avoid for manual commits):

```
WIP
Added new feature
fix bug
temp commit
```

### For Planning/Investigation

If you need to commit work-in-progress or planning notes, use:

```
chore(planning): initial investigation for feature X
docs(notes): document approach for issue #123
```

---

## Code Style & Standards

### TypeScript

- Use TypeScript for all new code
- Enable strict mode (`strict: true`)
- Prefer `interface` over `type` for object shapes
- Use explicit return types for public functions
- Avoid `any` - use `unknown` if type is truly unknown

### Naming Conventions

| Type       | Convention                       | Example               |
| ---------- | -------------------------------- | --------------------- |
| Files      | camelCase                        | `cookieJar.ts`        |
| Classes    | PascalCase                       | `CookieJar`           |
| Functions  | camelCase                        | `getCookieHeader`     |
| Constants  | UPPER_SNAKE_CASE                 | `MAX_RETRY_COUNT`     |
| Interfaces | PascalCase (optional `I` prefix) | `Cookie` or `ICookie` |
| Types      | PascalCase                       | `ProxyConfig`         |

### Code Organization

```
src/
├── core/           # Core utilities (http, auth, config)
├── features/       # Feature modules (leaderboard, share, etc.)
│   └── [feature]/
│       ├── index.ts    # Module exports
│       ├── tools.ts    # MCP tool definitions
│       ├── types.ts    # TypeScript types
│       └── api.ts      # API calls (if needed)
├── mcp-proxy/      # Proxy implementation
└── server.ts       # Main server entry
```

### Documentation

- Add JSDoc comments for all public APIs
- Use English for code comments
- Include `@param`, `@returns`, `@throws` tags
- Add `@example` for complex functions

```typescript
/**
 * Creates a fetch wrapper that manages cookies automatically.
 * @param cookieJar - The CookieJar instance to use
 * @returns A fetch-compatible function
 * @example
 * const jar = new CookieJar();
 * const customFetch = createCookieFetch(jar);
 * const response = await customFetch('https://api.example.com');
 */
export function createCookieFetch(cookieJar: CookieJar): typeof fetch {
  // ...
}
```

### Error Handling

- Use custom error classes for domain-specific errors
- Always include error context (what operation failed)
- Log errors with structured information
- Never swallow errors silently

```typescript
// ✅ Good
try {
  await client.connect();
} catch (error) {
  console.error('[Proxy] Connection failed:', formatError(error));
  throw error;
}

// ❌ Bad
try {
  await client.connect();
} catch (error) {
  // silently ignore
}
```

---


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [taptap/instant-games-open-mcp](https://github.com/taptap/instant-games-open-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-30 -->
