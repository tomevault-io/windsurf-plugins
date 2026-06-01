---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands
- Build: `npm run build` - Compiles TypeScript code to dist/
- Start: `npm run start` - Run the application with ts-node for development
- Test: `npm run test` - Run all Jest tests (configured to run serially to prevent file conflicts)
- Single test: `npx jest src/cli.test.ts -t "test name"` - Run specific test by name
- Update snapshots: `npm run test -- -u` - Update Jest snapshots after changes
- Format: `npm run format` - Format code with ESLint auto-fix
- Format (legacy): `npm run prettier` - Format code with Prettier (excludes .snap files)
- Lint: `npm run lint` - Check code with ESLint without fixing
- Publish: `npm run prepublishOnly` - Automatically runs build before publishing

## Token Analysis Scripts
- `npm run analyze-tokens` - Full analysis of Claude vs OpenAI token counts using Moby Dick
- `npm run calculate-multiplier` - Calculate the Claude to OpenAI token multiplier
- `npm run test-multiplier` - Test the current multiplier implementation with sample texts

## Version Management
When adding a new feature or making significant changes:
1. Bump the version in package.json following semantic versioning:
   - MAJOR version for incompatible API changes
   - MINOR version for new functionality in a backward compatible manner (e.g., 1.2.4 → 1.3.0)
   - PATCH version for backward compatible bug fixes
2. Run `npm install` to update package-lock.json
3. Document the new feature/change in CLAUDE.md with version number
4. Claude should automatically suggest version bumping when a new feature is added (only once per conversation)

## Project Architecture

This is a CLI tool called "ai-digest" that aggregates files into a single Markdown file for use with AI models. It operates both as a CLI tool (via Commander.js) and as a Node.js library with multiple export patterns.

### Dual CLI/Library Design
The architecture supports two usage patterns:
- **CLI Mode**: Command-line interface with file watching, progress display, and console output
- **Library Mode**: Programmatic API returning data structures for integration with other tools

### Core Processing Pipeline
1. **File Discovery**: Uses glob patterns to scan directories, respecting ignore patterns
2. **Binary Detection**: Separates text files from binary files using `isbinaryfile` and file extensions
3. **Content Processing**: Text files are wrapped in markdown code blocks; binary files get descriptive text
4. **Token Estimation**: Calculates token counts using Claude tokenizer with OpenAI estimation via multiplier
5. **Output Generation**: Combines processed files into a single markdown document

### Key Components
- `src/index.ts` - Main entry point containing CLI logic and library exports
- `src/digest.ts` - Core processing functions for file discovery, content processing, and output generation
- `src/utils.ts` - Utility functions for file processing, token counting, ignore patterns, and file type detection
- `src/types.ts` - TypeScript type definitions for the project
- `src/cli.test.ts` - CLI functionality tests (command-line interface behavior)
- `src/library.test.ts` - Core library function tests (generateDigest, generateDigestContent, etc.)
- `src/file-stats.test.ts` - File statistics tests (getFileStats function)
- `src/minify.test.ts` - Minify functionality tests (.aidigestminify patterns and callbacks)
- `scripts/` - Analysis and utility scripts for token counting research

### Critical Functions
- `processFiles()` - Core processing pipeline that handles file discovery, content processing, and statistics
- `generateDigestContent()` - Main library function that returns content + files + stats
- `generateDigest()` - Higher-level function for simple string output or file writing
- `generateDigestFiles()` - Returns array of processed file objects for custom processing
- `getFileStats()` - Returns file statistics sorted by processed content size with total token counts (added in v1.3.0)
- `writeDigestToFile()` - Handles file writing with progress display and statistics
- `watchFiles()` - Implements file watching with debouncing for auto-rebuild

### File Size Calculation Strategy
The tool consistently uses processed content size (markdown wrapper + content) for all file size calculations and displays. This ensures consistency between CLI output and library functions.

### Testing Architecture
- **Test Organization**: Tests are split into 4 files based on functionality:
  - `cli.test.ts` - CLI interface and command-line behavior
  - `library.test.ts` - Core library functions and API
  - `file-stats.test.ts` - File statistics and size calculation
  - `minify.test.ts` - Minify patterns and callback functionality
- **Serial Execution**: Jest is configured with `maxWorkers: 1` to prevent file conflicts during concurrent test execution
- **CLI Tests**: Use `execAsync` with `ts-node` to test actual CLI behavior
- **Library Tests**: Direct function imports for unit testing
- **Temporary Directories**: Each test creates isolated temp directories for file operations
- **Snapshot Testing**: Used for complex output structures (file stats, processed content)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [khromov/ai-digest](https://github.com/khromov/ai-digest) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
