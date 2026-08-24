---
trigger: always_on
description: This guide provides coding standards and best practices for developing JupyterLab extensions. Follow these rules to align with community standards and keep your extension maintainable.
---

# JupyterLab Extension Development

This guide provides coding standards and best practices for developing JupyterLab extensions. Follow these rules to align with community standards and keep your extension maintainable.

**Extension type**: frontend-and-server

## External Documentation and Resources

### PRIORITY RESOURCE USAGE

**When you encounter uncertainty, incomplete information, or need implementation examples, you MUST consult these external resources FIRST before attempting to implement features.**

Use your available tools (web search, documentation search) to access and retrieve content from these resources when:

- You're unsure about API usage, method signatures, or interface requirements
- You need to verify the correct approach for a feature or pattern
- You're looking for existing implementation examples or best practices
- You're debugging unexpected behavior and need official documentation
- You're implementing a feature that likely exists in core JupyterLab or other extensions

### Required External Resources

**These resources are PRIORITY references. Always check them when you need external information:**

1. **JupyterLab Extension Developer Guide**
   - URL: https://jupyterlab.readthedocs.io/en/stable/extension/extension_dev.html
   - Use for: Extension patterns, architecture overview, development workflow, and best practices
   - **Action**: Use web search or documentation tools to retrieve specific sections when needed

2. **JupyterLab API Reference (Frontend)**
   - URL: https://jupyterlab.readthedocs.io/en/latest/api/index.html
   - Use for: Complete API reference for all JupyterLab frontend packages, interfaces, classes, and methods
   - **Action**: Search for specific APIs when you need method signatures, interface definitions, or class documentation. For example, search "JupyterLab IRenderMime.IRenderer" or "JupyterLab ICommandPalette"

3. **JupyterLab Extension Examples Repository**
   - URL: https://github.com/jupyterlab/extension-examples
   - Use for: Working code examples, implementation patterns, complete working extensions
   - **Action**: Search this repository for similar features before implementing from scratch

4. **JupyterLab Core Repository**
   - URL: https://github.com/jupyterlab/jupyterlab
   - Use for: Reference implementations in `packages/` directory - all core packages are extensions themselves
   - **Action**: When implementing complex features, search this repo for how core extensions solve similar problems

5. **Jupyter Server API Documentation**
   - URL: https://jupyter-server.readthedocs.io/
   - Use for: Server-side API handlers, route setup, backend integration patterns
   - **Action**: Consult when working on backend routes or server extension configuration

6. **Project-Specific Documentation**
   - Locations: `README.md`, `RELEASE.md` in project root; check for `docs/` directory
   - Use for: Project requirements, specific configuration, custom conventions
   - **Action**: Read these files at the start of work and reference when making architectural decisions

### When to Use These Resources

**ALWAYS consult external documentation when:**

- ❗ You're about to implement a feature without knowing if there's an established pattern
- ❗ An API call or method isn't working as expected
- ❗ You need to understand the correct lifecycle methods or hooks
- ❗ You're uncertain about type definitions or interfaces
- ❗ You're implementing something that seems like it should be a common pattern

**HOW to access these resources:**

- 🔍 Use web search tools with specific queries like: "JupyterLab IRenderMime.IRenderer interface documentation"
- 🔍 Search GitHub repositories for code examples: "JupyterLab extension examples widget"
- 🔍 Retrieve documentation pages to read API specifications and usage guidelines
- 🔍 Look for working code in the extension-examples repository before writing custom implementations

**Remember:** These resources contain the authoritative information. Don't guess at API usage - look it up!

## Code Quality Rules

### Logging and Debugging

**❌ Don't**: Use `console.log()`
**✅ Do**: Use structured logging or user-facing notifications

```typescript
// In TypeScript files like src/index.ts
import { INotification } from '@jupyterlab/apputils';
app.commands.notifyCommandChanged();
```

**✅ Do**: Use `console.error()` to log low-level error details that should not be presented to users in the UI
**✅ Do**: Use `console.warn()` to log non-optimal conditions, e.g. an unexpected response from an external API that's been successfully handled.

### Type Safety

**✅ Do**: Define explicit interfaces (see example patterns in `src/index.ts`)

```typescript
interface PluginConfig {
  enabled: boolean;
  apiEndpoint: string;
}
```

**❌ Don't**: Use the `any` type in TypeScript files
**✅ Do**: Prefer typeguards over type casts

### File-Scoped Validation

After editing TypeScript files, run:

```bash
npx tsc --noEmit src/index.ts  # Check single file
npx tsc --noEmit               # Check all files
```

After editing Python files (like `jupyter_ai_acp_client/routes.py`):

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jupyter-ai-contrib/jupyter-ai-acp-client](https://github.com/jupyter-ai-contrib/jupyter-ai-acp-client) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-24 -->
