---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

**ccexp** (short for claude-code-explorer) - React Ink-based CLI tool for exploring and managing Claude Code settings and slash commands. The tool provides an interactive terminal UI for file navigation, content preview, and file management operations. The package was renamed from `claude-code-explorer` to `ccexp` for brevity and easier command-line usage.

## Core Commands

```bash
# Development
bun run start                   # Run CLI in development mode
bun run build                   # Build for production (outputs to dist/)
bun run typecheck              # TypeScript type checking

# Testing (InSource Testing Pattern)
bun run test                   # Run all tests
bun run test:watch            # Test in watch mode
bun run test src/_utils.ts    # Run tests for specific file

# Quality Management
bun run check                  # Biome lint/format check
bun run check:write           # Biome auto-fix
bun run check:unsafe          # Biome unsafe auto-fix
bun run knip                  # Check for unused dependencies/exports
bun run ci                    # Full CI pipeline (build + check + typecheck + knip + test)

# Release Management
bun run release               # Interactive version bumping with bumpp
bun run prepack              # Pre-publish build and package.json cleanup

# CLI Usage
./dist/index.js               # Interactive React Ink TUI mode
bun run start                 # Development mode with hot reload
bun run dev                   # Development mode with watch
```

## Technical Architecture

### Main Tech Stack

- **Runtime**: Bun + Node.js (>= 20) - ESM only
- **React TUI Framework**: React Ink (v6) for terminal UI components
- **UI Components**: @inkjs/ui for enhanced terminal components (TextInput, Spinner, StatusMessage)
- **Build**: tsdown (Rolldown/Oxc) → produces shebang executable with type definitions
  - Configured in `tsdown.config.ts` for ESM-only output
  - Automatic shebang injection for CLI usage
  - Type definition generation with `dts: true`
- **Testing**: vitest (InSource Testing + globals) + ink-testing-library
- **Linting**: Biome (v2.0.6) with strict rules
- **Dependency Management**: knip for unused dependency detection
- **File Operations**: fdir for fast directory scanning + node:fs/promises
- **Pattern Matching**: ts-pattern for complex conditional logic
- **Validation**: zod + branded types for runtime type safety
- **System Integration**: open, clipboardy for file operations

### Core Architecture Patterns

1. **InSource Testing**: Tests defined alongside source code for co-location

   ```typescript
   if (import.meta.vitest != null) {
     const { describe, test, expect } = import.meta.vitest;
     describe('functionName', () => {
       test('should work', () => {
         expect(result).toBe(expected);
       })
     })
   }
   ```

2. **Simplified Type System**: Clean types with runtime validation where needed

   ```typescript
   // Simple type alias approach
   export type ClaudeFilePath = string;

   // Runtime validation helper
   export const createClaudeFilePath = (path: string): ClaudeFilePath => {
     if (path.length === 0) {
       throw new Error('Path must not be empty');
     }
     return path;
   };
   ```

3. **React Ink Component Architecture**: React-based terminal UI

   ```typescript
   export function FileList({ files, onFileSelect }: FileListProps) {
     const [currentIndex, setCurrentIndex] = useState(0);
     const [isMenuMode, setIsMenuMode] = useState(false);

     useInput((input, key) => {
       // Handle keyboard navigation
     }, { isActive: !isMenuMode });

     return (
       <Box flexDirection="column">
         {/* File list UI */}
       </Box>
     );
   }
   ```

4. **Pattern Matching for File Type Detection**:

   ```typescript
   const detectClaudeFileType = (fileName: string, dirPath: string): ClaudeFileType => {
     return match([fileName, dirPath])
       .with(['CLAUDE.md', P._], () => 'project-memory' as const)
       .with(['CLAUDE.local.md', P._], () => 'project-memory-local' as const)
       .otherwise(() => 'unknown' as const);
   };
   ```

5. **React Ink Focus Management**: Proper input handling with `isActive` pattern

   ```typescript
   // FileList component
   useInput((input, key) => {
     if (key.upArrow) setCurrentIndex(prev => Math.max(0, prev - 1));
     if (key.downArrow) setCurrentIndex(prev => Math.min(files.length - 1, prev + 1));
     if (key.return) setIsMenuMode(true);
   }, { isActive: !isMenuMode });

   // MenuActions component
   useInput((input, key) => {
     if (key.escape) onClose();
     // Handle menu actions
   }, { isActive: true });
   ```

6. **Scanner Hierarchy**: Modular scanner architecture with base class

   ```typescript
   // Base scanner provides common functionality
   export abstract class BaseFileScanner<T> {
     protected abstract readonly maxFileSize: number;
     protected abstract readonly fileType: string;

     async processFile(filePath: string): Promise<T | null> {
       // Common file processing logic
     }

     protected abstract parseContent(
       filePath: string,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nyatinte/ccexp](https://github.com/nyatinte/ccexp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
