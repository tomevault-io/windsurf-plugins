---
trigger: always_on
description: > Instructions for GitHub Copilot, OpenAI Codex, and other AI coding assistants working on the tuck project.
---

# AGENTS.md - AI Coding Assistant Guidelines for tuck

> Instructions for GitHub Copilot, OpenAI Codex, and other AI coding assistants working on the tuck project.

## Project Overview

**tuck** is a modern dotfiles manager CLI built with TypeScript and Node.js. It provides a beautiful, safe, and git-native way to manage configuration files across machines.

### Core Values

1. **Safety First** — Never lose user data, always confirm destructive operations
2. **Beautiful by Default** — Polished terminal UI using @clack/prompts, chalk, boxen
3. **Git-Native** — Built on git but abstracts complexity
4. **Zero-Config Start** — Works immediately, powerful when configured

---

## Tech Stack Reference

| Component | Technology | Notes |
|-----------|------------|-------|
| Runtime | Node.js 18+ | ESM modules only |
| Language | TypeScript 5.x | Strict mode enabled |
| Package Manager | pnpm 9+ | Use `pnpm` not `npm` |
| CLI Framework | Commander.js | v11+ |
| Prompts | @clack/prompts | For interactive UI |
| Styling | chalk, boxen, ora | Terminal colors/boxes/spinners |
| Git | simple-git | Async git operations |
| File System | fs-extra | Extended fs operations |
| Validation | Zod | Runtime type validation |
| Testing | Vitest | Unit and integration tests |
| Build | tsup | Bundle to single file |

---

## Directory Structure

```
tuck/
├── src/
│   ├── commands/        # CLI command implementations
│   │   ├── init.ts      # tuck init - Initialize tuck
│   │   ├── add.ts       # tuck add <path> - Track a file
│   │   ├── remove.ts    # tuck remove <path> - Untrack a file
│   │   ├── sync.ts      # tuck sync - Sync changes
│   │   ├── push.ts      # tuck push - Push to remote
│   │   ├── pull.ts      # tuck pull - Pull from remote
│   │   ├── restore.ts   # tuck restore - Restore from backup
│   │   ├── status.ts    # tuck status - Show status
│   │   ├── list.ts      # tuck list - List tracked files
│   │   ├── diff.ts      # tuck diff - Show differences
│   │   ├── config.ts    # tuck config - Manage configuration
│   │   ├── apply.ts     # tuck apply - Apply dotfiles from repo
│   │   ├── undo.ts      # tuck undo - Undo/restore snapshots
│   │   └── scan.ts      # tuck scan - Detect dotfiles on system
│   ├── lib/             # Core modules
│   │   ├── paths.ts     # Path utilities
│   │   ├── config.ts    # Config management
│   │   ├── manifest.ts  # File tracking
│   │   ├── git.ts       # Git wrapper
│   │   ├── files.ts     # File operations
│   │   ├── backup.ts    # Backup system
│   │   ├── hooks.ts     # Lifecycle hooks
│   │   ├── github.ts    # GitHub CLI integration
│   │   ├── timemachine.ts # Snapshot/time-machine backups
│   │   ├── merge.ts     # Smart merging for shell files
│   │   └── detect.ts    # Dotfile detection
│   ├── ui/              # UI components
│   │   ├── banner.ts    # ASCII art
│   │   ├── logger.ts    # Styled logs
│   │   ├── prompts.ts   # Interactive prompts
│   │   ├── spinner.ts   # Loading spinners
│   │   └── table.ts     # Table output
│   ├── schemas/         # Zod schemas
│   │   ├── config.schema.ts   # Configuration schema
│   │   └── manifest.schema.ts # Manifest schema
│   ├── constants.ts     # App constants
│   ├── types.ts         # TypeScript types
│   ├── errors.ts        # Custom errors
│   └── index.ts         # Entry point
├── tests/               # Test files
├── dist/                # Build output
└── docs/                # Documentation
```

---

## Code Generation Guidelines

### DO Generate

1. **Explicit TypeScript types**
   ```typescript
   // Good
   interface FileChange {
     path: string;
     status: 'added' | 'modified' | 'deleted';
     checksum: string;
   }

   const detectChanges = async (dir: string): Promise<FileChange[]> => {
     // implementation
   };
   ```

2. **Proper error handling with custom errors**
   ```typescript
   // Good
   import { FileNotFoundError, PermissionError } from '../errors.js';

   if (!await pathExists(filePath)) {
     throw new FileNotFoundError(filePath);
   }
   ```

3. **User feedback with UI utilities**
   ```typescript
   // Good
   import { prompts, logger } from '../ui/index.js';

   prompts.intro('tuck sync');
   const spinner = prompts.spinner();
   spinner.start('Syncing files...');
   // work
   spinner.stop('Synced 5 files');
   prompts.outro('Done!');
   ```

4. **Confirmation for destructive actions**
   ```typescript
   // Good
   const confirmed = await prompts.confirm(
     'This will overwrite existing files. Continue?',
     false
   );
   if (!confirmed) {
     prompts.cancel('Operation cancelled');
     return;
   }
   ```

5. **Path handling with utilities**
   ```typescript
   // Good
   import { expandPath, collapsePath, pathExists } from '../lib/paths.js';

   const fullPath = expandPath('~/.zshrc');
   const displayPath = collapsePath(fullPath);
   ```

### DO NOT Generate

1. **Never use `any` type**
   ```typescript
   // Bad
   const data: any = JSON.parse(content);

   // Good
   const data: unknown = JSON.parse(content);
   const validated = ConfigSchema.parse(data);
   ```

2. **Never ignore errors silently**
   ```typescript
   // Bad
   await copyFile(src, dest).catch(() => {});

   // Good
   try {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Pranav-Karra-3301/tuck](https://github.com/Pranav-Karra-3301/tuck) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
