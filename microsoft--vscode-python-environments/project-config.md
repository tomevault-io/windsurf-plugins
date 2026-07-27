---
trigger: always_on
description: This guide provides comprehensive instructions for AI agents on the complete testing workflow: writing tests, running them, diagnosing failures, and fixing issues. Use this guide whenever working with test files or when users request testing tasks.
---


# AI Testing Workflow Guide: Write, Run, and Fix Tests

This guide provides comprehensive instructions for AI agents on the complete testing workflow: writing tests, running them, diagnosing failures, and fixing issues. Use this guide whenever working with test files or when users request testing tasks.

## Complete Testing Workflow

This guide covers the full testing lifecycle:

1. **📝 Writing Tests** - Create comprehensive test suites
2. **▶️ Running Tests** - Execute tests using VS Code tools
3. **🔍 Diagnosing Issues** - Analyze failures and errors
4. **🛠️ Fixing Problems** - Resolve compilation and runtime issues
5. **✅ Validation** - Ensure coverage and resilience

### When to Use This Guide

**User Requests Testing:**

- "Write tests for this function"
- "Run the tests"
- "Fix the failing tests"
- "Test this code"
- "Add test coverage"

**File Context Triggers:**

- Working in `**/test/**` directories
- Files ending in `.test.ts` or `.unit.test.ts`
- Test failures or compilation errors
- Coverage reports or test output analysis

## 🚨 CRITICAL: Common Mistakes (Read First!)

These mistakes have occurred REPEATEDLY. Check this list BEFORE writing any test code:

| Mistake                                        | Fix                                                                                        |
| ---------------------------------------------- | ------------------------------------------------------------------------------------------ |
| Hardcoded POSIX paths like `'/test/workspace'` | Use `'.'` for relative paths, `Uri.file(x).fsPath` for comparisons                         |
| Stubbing `workspace.getConfiguration` directly | Stub the wrapper `workspaceApis.getConfiguration` instead                                  |
| Stubbing `workspace.workspaceFolders` property | Stub wrapper function `workspaceApis.getWorkspaceFolders()`                                |
| Comparing `fsPath` to raw string               | Compare `fsPath` to `Uri.file(expected).fsPath`                                            |
| Stubbing `commands.executeCommand` directly    | First update production code to use `executeCommand` from `command.api.ts`, then stub that |
| Stubbing `window.createTreeView` directly      | First update production code to use `createTreeView` from `window.apis.ts`, then stub that |

**Pre-flight checklist before completing test work:**

- [ ] All paths use `Uri.file().fsPath` (no hardcoded `/path/to/x`)
- [ ] All VS Code API stubs use wrapper modules, not `vscode.*` directly
- [ ] Production code uses wrappers for any VS Code API that tests need to stub (check `src/common/*.apis.ts`)
- [ ] Tests pass on both Windows and POSIX

## Test Types

When implementing tests as an AI agent, choose between two main types:

### Unit Tests (`*.unit.test.ts`)

- **Fast isolated testing** - Mock all external dependencies
- **Use for**: Pure functions, business logic, data transformations
- **Execute with**: `runTests` tool with specific file patterns
- **Mock everything** - VS Code APIs automatically mocked via `/src/test/unittests.ts`

### Extension Tests (`*.test.ts`)

- **Full VS Code integration** - Real environment with actual APIs
- **Use for**: Command registration, UI interactions, extension lifecycle
- **Execute with**: VS Code launch configurations or `runTests` tool
- **Slower but comprehensive** - Tests complete user workflows

## 🤖 Agent Tool Usage for Test Execution

### Primary Tool: `runTests`

Use the `runTests` tool to execute tests programmatically rather than terminal commands for better integration and result parsing:

```typescript
// Run specific test files
await runTests({
    files: ['/absolute/path/to/test.unit.test.ts'],
    mode: 'run',
});

// Run tests with coverage
await runTests({
    files: ['/absolute/path/to/test.unit.test.ts'],
    mode: 'coverage',
    coverageFiles: ['/absolute/path/to/source.ts'],
});

// Run specific test names
await runTests({
    files: ['/absolute/path/to/test.unit.test.ts'],
    testNames: ['should handle edge case', 'should validate input'],
});
```

### Compilation Requirements

Before running tests, ensure compilation. Always start compilation with `npm run watch-tests` before test execution to ensure TypeScript files are built. Recompile after making import/export changes before running tests, as stubs won't work if they're applied to old compiled JavaScript that doesn't have the updated imports:

```typescript
// Start watch mode for auto-compilation
await run_in_terminal({
    command: 'npm run watch-tests',
    isBackground: true,
    explanation: 'Start test compilation in watch mode',
});

// Or compile manually
await run_in_terminal({
    command: 'npm run compile-tests',
    isBackground: false,
    explanation: 'Compile TypeScript test files',
});
```

### Alternative: Terminal Execution


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/vscode-python-environments](https://github.com/microsoft/vscode-python-environments) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
