---
trigger: always_on
description: **Single source of truth → Compile to multiple formats → Diff-based sync**
---

# vsync - AI Coding Tool Config Synchronizer

**Single source of truth → Compile to multiple formats → Diff-based sync**

Syncs Skills, MCP, Agents & Commands across Claude Code, Cursor, OpenCode, Codex.

---

## 🎯 Core Concept

**NOT**: File copy tool
**IS**: Config compiler + diff executor

```
Source (Claude Code) → Normalize → Diff → Plan → Compile → Atomic Write → Update Manifest
```

---

## 🚨 Critical Rules (Read FIRST)

### 1. Follow Documentation Order

```
Before ANY work:
1. Read TASKS.md → Find current phase & task
2. Read docs/prd.md Section 2 → Config formats (MOST IMPORTANT!)
3. Follow TDD → Write tests FIRST
```

### 2. Config Format Precision (See PRD Section 2.2)

```typescript
// ⚠️ CRITICAL: Each tool has DIFFERENT formats

// OpenCode
{ "mcp": { "type": "stdio", "environment": { "X": "{env:VAR}" } } }
//  ^^^      ^^^^^^                          ^^^^^^^^^^^
//  NOT mcpServers!  Required!               Curly braces!

// Claude Code
{ "mcpServers": { "env": { "X": "${VAR}" } } }
//              No type field    ^^^^^^^
//                              Dollar sign, no env: prefix

// Cursor
{ "mcpServers": { "env": { "X": "${env:VAR}" } } }
//              No type field    ^^^^^^^^^^^^
//                              Dollar sign WITH env: prefix
```

### 3. Environment Variables

```typescript
// ❌ WRONG - Expands variables
const config = { env: { TOKEN: process.env.GITHUB_TOKEN } };

// ✅ CORRECT - Preserves syntax
const config = { env: { TOKEN: "${env:GITHUB_TOKEN}" } };
```

### 4. File Operations

```typescript
// ❌ WRONG - Not crash-safe
await fs.writeFile(path, content);

// ✅ CORRECT - Atomic write
import { atomicWrite } from "./utils/atomic-write";
await atomicWrite(path, content);
```

### 5. Development Workflow

```
1. TASKS.md → Pick task
2. Write TEST first (TDD)
3. Implement code
4. pnpm test && pnpm typecheck && pnpm lint
5. Mark [x] in TASKS.md
6. Commit (Angular convention)
```

---

## 📚 Documentation Map

| Document                            | When to Read      | Contains                                |
| ----------------------------------- | ----------------- | --------------------------------------- |
| **TASKS.md**                        | Before every task | Current phase, task breakdown, progress |
| **docs/prd.md**                     | When implementing | **Complete spec, config formats**       |
| **docs/config.md**                  | When needed       | Project configuration values            |
| **.claude/commands/do-task.md**     | Shortcut          | TDD workflow, patterns, validation      |
| **.claude/commands/create-task.md** | Shortcut          | Task creation templates                 |

**PRD Section Quick Lookup:**

- **Section 2** (Config Formats) ← **READ THIS BEFORE ADAPTERS**
- Section 4 (CLI Commands) ← Before CLI work
- Section 5 (Security) ← Before security work
- Section 6 (Adapter Architecture) ← Before adapter design
- Section 7 (Diff & Plan) ← Before diff/plan work
- Section 8 (MVP Roadmap) ← For scope validation

---

## 🚀 Quick Start Commands

**Execute tasks:**

```bash
/do-task next         # Start next task
/do-task Phase 1.1    # Specific task
```

**Create tasks:**

```bash
/create-task feature Add feature name
/create-task bugfix Fix issue description
```

These commands auto-read TASKS.md, PRD, and guide TDD workflow.

---

## 🎯 Current Scope (See TASKS.md for phases)

**MVP (v1.0) - COMPLETE:**

- ✅ Skills + MCP sync
- ✅ Claude Code (source) → Cursor, OpenCode (targets)
- ✅ Safe/Prune modes
- ✅ Project-level config
- ✅ Atomic writes

**v1.1 - COMPLETE:**

- ✅ Agents + Commands sync
- ✅ Codex adapter
- ✅ User-level config

**v1.2 - IN PROGRESS:**

- 🟡 Performance optimization
- 🟡 Watch mode
- 🟡 GitHub Action integration

---

## ⚠️ Common Mistakes (Full list in PRD Section 2)

| ❌ Wrong                     | ✅ Correct             |
| ---------------------------- | ---------------------- |
| OpenCode: `mcpServers`       | `mcp`                  |
| OpenCode: no `type`          | `type: "local/remote"` |
| OpenCode: `${env:VAR}`       | `{env:VAR}`            |
| Claude Code: `${env:VAR}`    | `${VAR}`               |
| Cursor: `${VAR}` (uppercase) | `${env:VAR}`           |
| `fs.writeFile()`             | `atomicWrite()`        |
| Implement first              | Test first (TDD)       |

---

## 🔄 Automation Preferences

**DO automatically:**

- Run tests/typecheck/lint
- Update TASKS.md progress
- Create commits (Angular convention)
- Install listed dependencies

**ASK before:**

- Adding unlisted dependencies
- Changing PRD specifications
- Skipping phases
- Force operations

---

## 📦 Tech Stack & Project Structure

**Architecture**: pnpm monorepo

```
vsync/                  # Project root
├── pnpm-workspace.yaml     # Workspace config (root level)
├── cli/                    # CLI workspace package
│   ├── package.json        # CLI dependencies
│   ├── tsconfig.json       # TypeScript config
│   ├── src/                # CLI source code
│   │   ├── cli/            # CLI commands
│   │   ├── core/           # Core logic
│   │   ├── adapters/       # Tool adapters
│   │   ├── types/          # TypeScript types
│   │   ├── utils/          # Utilities
│   │   └── index.ts        # Entry point

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nicepkg/vsync](https://github.com/nicepkg/vsync) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
