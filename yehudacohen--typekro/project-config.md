---
trigger: always_on
description: - **Build**: `bun run build` (full build with lint/typecheck)
---

# ⚠️ CRITICAL: NEVER CREATE BACKUP FILES (.backup, .old, -backup.ts, etc.) - USE GIT FOR VERSION CONTROL!
# ⚠️ CRITICAL: NO DESTRUCTIVE GIT COMMANDS WITHOUT EXPLICIT USER PERMISSION!
#
# AGENT GUIDELINES - READ BEFORE MAKING ANY CHANGES

## QUICK REFERENCE (20-Line Summary)

### Build/Test Commands
- **Build**: `bun run build` (full build with lint/typecheck)
- **Test single file**: `bun test test/path/to/file.test.ts --timeout 10000`
- **Test unit only**: `bun run test` (excludes integration tests)
- **Test integration**: `bun run test:integration`
- **Test with coverage**: `bun run test:coverage` (generates lcov report)
- **Lint**: `bun run lint` (check) / `bun run lint:fix` (auto-fix)
- **Typecheck**: `bun run typecheck` (all) / `bun run typecheck:lib` (src only)
- **Format**: `bun run format:fix`

### Code Style
- **Tool**: Use `bun` (never npm/yarn). Biome for linting/formatting.
- **Imports**: External libraries first, internal modules second, types last. Use `type` imports for types (enforced by Biome).
- **Format**: 2 spaces, single quotes, 100 char lines, trailing commas ES5, semicolons always
- **Types**: Strict TypeScript. No `as any` except in specific core files. No `!` assertions except tests.
- **Type Safety**: Arrays/objects require null checks (`arr[0]?.value`). Optional properties are exact - no `undefined` when not specified.
- **Naming**: camelCase variables/functions, PascalCase types/interfaces, UPPER_SNAKE constants
- **Errors**: Custom error classes extending `TypeKroError`. Use structured error messages.
- **Documentation**: Use JSDoc comments for public APIs, exported functions, and complex types.
- **Patterns**: Use `createResource` pattern for internal factories. Follow `src/core/`, `src/factories/` structure.

### Which Composition API to Use
- **Default: `kubernetesComposition`** — single function creates resources + returns status. Status expressions are natural JavaScript, auto-converted to CEL. Use `simple.Deployment()`, `simple.Service()`, etc.
- **Advanced: `toResourceGraph`** — separate resource builder and status builder. Status uses explicit `Cel.expr()` / `Cel.template()`. Use `createDeployment()`, `createService()`, etc.
- **Rule of thumb**: Start with `kubernetesComposition`. Switch to `toResourceGraph` only if you need explicit CEL control.

### kubernetesComposition Patterns (Recommended)
- **Imports**: `import { kubernetesComposition, simple } from 'typekro';` (or `from '../src/index.js'`)
  // Note: simple.* factories are also importable individually from 'typekro/simple'
- **Resources**: `const deploy = simple.Deployment({ id: 'deploy', name: spec.name, image: spec.image })`
- **Status**: Return natural JS: `return { ready: deploy.status.readyReplicas >= spec.replicas }`
- **String templates**: `` url: `http://${svc.status.clusterIP}` ``
- **Conditionals**: `phase: deploy.status.readyReplicas > 0 ? 'running' : 'pending'`
- **`id` parameter**: Required on every resource for cross-resource references

### toResourceGraph Patterns (Advanced)
- **Use proper CEL expressions**: `Cel.expr<Type>(resourceRef, operator)` or `Cel.expr<Type>('static_expression')`
- **Status expressions**: `Cel.expr<boolean>(resources.myResource.status.field, ' == "Ready"')` (NO `?.` — Enhanced types are NonOptional)
- **Multi-condition**: `Cel.expr<string>('condition ? "value1" : "value2"')`  
- **Templates**: `Cel.template('Hello %s', schema.spec.name)` for string interpolation
- **Resource references**: Use `resources.resourceKey.status.field` (NO `?.` — Enhanced types are NonOptional)
- **Type safety**: Always specify CEL type: `Cel.expr<boolean>()`, `Cel.expr<string>()`, etc.

## CRITICAL RULES - NEVER VIOLATE THESE

### 1. NO BACKUP FILES OR WORKSPACE POLLUTION
- **NEVER** create `.backup`, `.backup2`, `.old`, etc. files anywhere in the workspace
- **NEVER** create temporary files in the workspace root (debug-*.js, test-*.ts, temp-*.md)
- **NEVER** create duplicate files with slight name variations (-enhanced.ts, -improved.ts, -timeout-fix.ts)
- Use git for version control, not manual backups
- Use `/tmp` or ask where to put temporary files
- Clean up after yourself in the same session

### 2. NO DESTRUCTIVE CHANGES WITHOUT PERMISSION - ESPECIALLY NO WORKING DIRECTORY DELETION
- **NEVER EVER** run `git checkout HEAD --`, `git checkout --`, `git reset --hard`, or any command that deletes working directory changes
- **THESE COMMANDS DESTROY USER'S UNCOMMITTED WORK** - this is the worst possible mistake an agent can make
- **WORKING DIRECTORY CHANGES ARE SACRED** - they represent the user's current progress and should never be touched
- **NEVER** delete files without asking first
- **NEVER** revert changes blindly without understanding what they contain
- **ALWAYS** check what changes contain before modifying them
- **ALWAYS** ask "What will this command do?" before running destructive git commands
- **IF YOU NEED TO UNDO YOUR CHANGES**: Make targeted edits to fix them, don't use destructive git commands

### 2.1. NEVER COMMIT USER'S UNCOMMITTED CHANGES WITHOUT PERMISSION
- **CRITICAL**: When user asks to commit specific changes, ONLY commit those exact changes
- **NEVER** run `git add .` or `git commit -a` which commits ALL changes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yehudacohen/typekro](https://github.com/yehudacohen/typekro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
