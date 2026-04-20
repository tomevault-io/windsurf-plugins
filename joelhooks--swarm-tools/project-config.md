---
trigger: always_on
description: **The plugin wrapper at `~/.config/opencode/plugin/swarm.ts` must have ZERO imports from `opencode-swarm-plugin`.**
---

# Monorepo Guide: Bun + Turborepo

## CRITICAL: Plugin Wrapper Must Be Self-Contained

**The plugin wrapper at `~/.config/opencode/plugin/swarm.ts` must have ZERO imports from `opencode-swarm-plugin`.**

```
╔═══════════════════════════════════════════════════════════════════════════╗
║                                                                           ║
║   ❌ NEVER DO THIS IN THE PLUGIN WRAPPER:                                 ║
║                                                                           ║
║   import { anything } from "opencode-swarm-plugin";  // BREAKS OPENCODE   ║
║   import { stuff } from "swarm-mail";                // BREAKS OPENCODE   ║
║                                                                           ║
║   ✅ ONLY THESE IMPORTS ARE SAFE:                                         ║
║                                                                           ║
║   import { ... } from "@opencode-ai/plugin";  // Provided by OpenCode     ║
║   import { ... } from "@opencode-ai/sdk";     // Provided by OpenCode     ║
║   import { ... } from "node:*";               // Node.js builtins         ║
║                                                                           ║
╚═══════════════════════════════════════════════════════════════════════════╝
```

**Why?** The npm package has transitive dependencies (`evalite`, etc.) that aren't available in OpenCode's plugin context. Importing causes: `Cannot find module 'evalite/runner'` → trace trap → OpenCode crash.

**Pattern: Plugin wrapper is DUMB, CLI is SMART.**
- **Wrapper:** Thin shell, no logic, just bridges to `swarm` CLI via `spawn()`
- **CLI:** All the smarts, all the deps, runs in its own Node.js context

**If you need logic in the wrapper:** INLINE IT. Copy the code directly into the template. See the `// Swarm Signature Detection (INLINED)` section for an example of ~250 lines of inlined logic.

**Template location:** `packages/opencode-swarm-plugin/examples/plugin-wrapper-template.ts`

---

## CRITICAL: No `bd` CLI Commands

**NEVER use `bd` CLI commands in code.** The `bd` CLI is deprecated and should not be called via `Bun.$` or any shell execution.

Instead, use the **HiveAdapter** from `swarm-mail` package:

```typescript
import { createHiveAdapter } from "swarm-mail";

const adapter = await createHiveAdapter({ projectPath: "/path/to/project" });

// Query cells
const cells = await adapter.queryCells({ status: "open" });

// Create cell
const cell = await adapter.createCell({ title: "Task", type: "task" });

// Update cell
await adapter.updateCell(cellId, { description: "Updated" });

// Close cell
await adapter.closeCell(cellId, "Done");
```

**Why?** The `bd` CLI requires a separate installation and isn't available in all environments. The HiveAdapter provides the same functionality programmatically with proper TypeScript types.

---

## CRITICAL: Single Global Database Architecture

**All swarm data lives in ONE database: `~/.config/swarm-tools/swarm.db`**

```
╔═══════════════════════════════════════════════════════════════════════════╗
║                                                                           ║
║   ✅ GLOBAL DATABASE (the only one):                                      ║
║      ~/.config/swarm-tools/swarm.db                                       ║
║                                                                           ║
║   ❌ LOCAL DATABASES (banned, auto-migrated):                             ║
║      .opencode/swarm.db                                                   ║
║      .hive/swarm-mail.db                                                  ║
║      packages/*/.opencode/swarm.db                                        ║
║                                                                           ║
╚═══════════════════════════════════════════════════════════════════════════╝
```

### Why Single Database?

1. **No stray data** - All events, beads, messages in one place
2. **Cross-project visibility** - `swarm stats` shows everything
3. **Simpler debugging** - One database to inspect
4. **No migration headaches** - Data doesn't get lost in project-local DBs

### Runtime Guard

The `getOrCreateAdapter()` function in `swarm-mail` has a runtime guard that throws if code attempts to create a non-global database:

```typescript
// This will THROW:
const adapter = await createLibSQLAdapter({ url: "file:./local.db" });

// This is CORRECT (uses global path automatically):
const swarmMail = await getSwarmMailLibSQL();
```

### Auto-Migration

When `swarm setup` runs, it:
1. Detects stray databases in `.opencode/`, `.hive/`, `packages/*/`
2. Migrates unique data to global database (INSERT OR IGNORE)
3. Renames strays to `*.db.migrated` to prevent re-migration

### For Tests Only

Tests can use in-memory databases:

```typescript
// ✅ CORRECT for tests
const swarmMail = await createInMemorySwarmMailLibSQL("test-123");

// ❌ WRONG - don't create file-based test DBs
const adapter = await createLibSQLAdapter({ url: "file:./test.db" });
```

### Audit Report

See `.hive/analysis/stray-database-audit.md` for the full audit of database paths in the codebase.

---

## Prime Directive: TDD Everything


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [joelhooks/swarm-tools](https://github.com/joelhooks/swarm-tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
