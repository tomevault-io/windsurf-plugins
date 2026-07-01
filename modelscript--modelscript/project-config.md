---
trigger: always_on
description: **IMPORTANT:** On this system, `node -e` with multi-line inline scripts causes the terminal to hang indefinitely even after the command finishes. This is a known Antigravity bug on Linux.
---

# ModelScript Agent Guidelines

## Terminal Workaround (Linux)

**IMPORTANT:** On this system, `node -e` with multi-line inline scripts causes the terminal to hang indefinitely even after the command finishes. This is a known Antigravity bug on Linux.

**Rules:**

1. **Never** use `node -e "..."` with multi-line JavaScript/TypeScript.
2. Instead, write the script to a temp file and run it:
   ```bash
   cat > /tmp/script.ts << 'EOF'
   // your code here
   EOF
   npx tsx /tmp/script.ts
   ```
3. For simple one-liners, prefer `npm run test` or other package scripts.
4. Always set `WaitMsBeforeAsync` appropriately — use short timeouts (500-2000ms) for commands that might hang, and check with `command_status`.
5. **Build commands** (`npm run build`, `nx run`, etc.) hang indefinitely when there are lint or compilation errors. Always use a short `WaitMsBeforeAsync` (e.g., 500ms) and monitor with `command_status`, or pipe through `timeout 30` to force termination.
6. **Command Canceled:** Always query the output of a command using `command_status` with `WaitDurationSeconds: 0` even if its status was previously reported as `CANCELED` during a poll. A canceled command might still have produced useful error output before being terminated.
7. **Apparent hangs:** When a command appears to hang (stuck in RUNNING with no output), **always assume it is the known Antigravity Linux terminal bug first**, not an infinite loop in the code. Query `command_status` with `WaitDurationSeconds: 0` and `OutputCharacterCount` to check if output was already produced. Do not prematurely terminate commands or assume code bugs without first verifying the output.
8. **Test script placement:** Never place test scripts that import project modules in `/tmp/`. Scripts in `/tmp/` cannot resolve monorepo workspace imports (e.g., `@modelscript/core`) or relative paths back into the repo. Always place ad-hoc test scripts inside the repo (e.g., `packages/core/tests/` or a scratch file alongside the source).
9. **Tree-sitter for Node.js:** When writing scripts that run on desktop/Node.js (tests, CLI tools, etc.), always use the **native** `tree-sitter` package — never `web-tree-sitter` or WASM. Follow the pattern in `packages/core/tests/jest.setup.ts`:
   ```typescript
   import Modelica from "@modelscript/tree-sitter-modelica";
   import Parser from "tree-sitter";
   import { Context } from "../src/compiler/context.js";
   const parser = new Parser();
   parser.setLanguage(Modelica);
   Context.registerParser(".mo", parser);
   ```
10. **SafeToAutoRun:** Always set `SafeToAutoRun` to `false` for test scripts, debug scripts, and any command whose output you need to read. Setting it to `true` causes the command to run in the background where it appears frozen. Only use `SafeToAutoRun: true` for trivial commands like `cat`, `ls`, or `echo` that complete almost instantly.

## Concrete Syntax Nodes

**IMPORTANT:** Never use concrete syntax nodes (tree-sitter `SyntaxNode`) for flattening or interpretation logic. Concrete syntax nodes are ephemeral — they are only available during initial parsing and are NOT preserved through cloning, modification merging, or serialization.

**Rules:**

1. **Always use abstract syntax nodes** (`ModelicaClassDefinitionSyntaxNode`, `ModelicaLongClassSpecifierSyntaxNode`, etc.) for accessing class elements, sections, and other structural information in the flattener, interpreter, and model.
2. **Never access `concreteSyntaxNode` fields** for semantic analysis. They may be `null` on cloned or deserialized instances.
3. Properties like `sections`, `elements`, `equations`, `statements` are populated at construction time from whichever source is available (concrete or abstract). After construction, always access them through the abstract syntax node wrappers.
4. In the arena-native pipeline (`ArenaQueryFlattener`), CST nodes are retrieved via `db.cstNode(classId)` and **immediately** wrapped in AST nodes: `ModelicaClassDefinitionSyntaxNode.new(null, cstNode)`. Never hold raw CST references across function boundaries.

## Compilation Pipeline Architecture

There are **two flattening pipelines** in the codebase:

| Pipeline         | Entry Point                                      | Status                    | Output                               |
| ---------------- | ------------------------------------------------ | ------------------------- | ------------------------------------ |
| **Legacy**       | `Context.flattenDAE()` → `ModelicaFlattener`     | Full-featured, 12k+ lines | `ModelicaDAE` object graph           |
| **Arena-native** | `Context.flattenArena()` → `ArenaQueryFlattener` | New canonical pipeline    | `ArenaDAEBuilder` (struct-of-arrays) |

The arena-native pipeline is the target architecture. All new work should go into the arena pipeline unless fixing a bug in the legacy flattener.

### Pipeline Stages

```
Source (.mo) → tree-sitter → CST
    → SymbolIndexer → SymbolIndex (flat symbol table)
    → WorkspaceIndex → Unified SymbolIndex (multi-file)
    → QueryEngine (Salsa memoization)
    → Linting (lint__ query hooks)
    → ArenaQueryFlattener → ArenaDAEBuilder

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [modelscript/modelscript](https://github.com/modelscript/modelscript) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-01 -->
