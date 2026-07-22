---
trigger: always_on
description: vscode-icons is a VS Code (and Zed editor) extension providing 1400+ file and folder icons. The primary contribution type is adding new icons or extending existing icon associations.
---

# AGENTS.md - AI Agent Instructions for vscode-icons

## Project Overview

vscode-icons is a VS Code (and Zed editor) extension providing 1400+ file and folder icons. The primary contribution type is adding new icons or extending existing icon associations.

## Quick Reference

| Task | Files to Change |
|------|----------------|
| New file icon | `icons/file_type_{name}.svg` + `src/iconsManifest/supportedExtensions.ts` |
| New folder icon | `icons/folder_type_{name}.svg` + `icons/folder_type_{name}_opened.svg` + `src/iconsManifest/supportedFolders.ts` |
| New language ID | `src/iconsManifest/languages.ts` + `src/iconsManifest/supportedExtensions.ts` |
| Add extension to existing icon | `src/iconsManifest/supportedExtensions.ts` (modify existing entry) |

## Tech Stack

- **Language**: TypeScript (ES6/CommonJS)
- **DI**: inversify
- **Tests**: Mocha + Chai + Sinon
- **Bundler**: Webpack
- **Linting**: ESLint + Prettier
- **Node**: >= 18.15.0

## Setup & Verification

```bash
npm install                # Install dependencies
npm test                   # Compile + run tests (validates all icon entries)
npm run lint               # ESLint check
npm run format             # Prettier + ESLint fix
npm run compile:dev        # Compile TypeScript (dev, with tests)
npm run build:dev          # Compile + generate icon manifests
```

## Code Style

- Single quotes, trailing commas (always-multiline), arrow parens as-needed
- Max line length: 120 characters
- Interfaces prefixed with `I` (e.g., `IFileExtension`)
- Explicit member accessibility (public/private/protected)
- No `console.log` (use `console.info` or `console.error`)
- Pre-commit hooks enforce Prettier + ESLint via lint-staged

## Project Structure

```
icons/                              # SVG icon files (~1475 files)
src/
  iconsManifest/
    supportedExtensions.ts          # File icon definitions (main config)
    supportedFolders.ts             # Folder icon definitions
    languages.ts                    # Language ID mappings
    manifestBuilder.ts              # Builds VSCode + Zed manifests
    iconsGenerator.ts               # Orchestrates manifest generation
    customsMerger.ts                # Merges user customizations
  models/extensions/
    fileExtension.ts                # IFileExtension interface
    folderExtension.ts              # IFolderExtension interface
    fileFormat.ts                   # FileFormat enum (svg, png, etc.)
  models/language/language.ts       # ILanguage interface
test/                               # Test files (mirrors src/)
  iconsManifest/
    specsExtensions.test.ts         # Validates all file icon entries
    specsFolders.test.ts            # Validates all folder icon entries
```

### Key File Landmarks

- **`src/iconsManifest/supportedExtensions.ts`**: The `supported` array starts at **line 8**. Entries are alphabetically ordered by `icon` name (first entry: `access`, last: ~`knip`). New entries must be inserted in alphabetical order within this array.
- **`src/iconsManifest/supportedFolders.ts`**: The `supported` array starts at **line 8**. Entries are alphabetically ordered by `icon` name (first: `android`, last: `zed`).
- **`src/iconsManifest/languages.ts`**: The languages object starts at **line 3**. Entries are alphabetically ordered by key (first: `actionscript`, last: `zip`).

## Adding a File Icon

1. Create SVG icon at `icons/file_type_{name}.svg`
2. Add entry to `src/iconsManifest/supportedExtensions.ts` in **alphabetical order**:

```typescript
// By filename (e.g., config files):
{ icon: 'name', extensions: ['.configrc'], filename: true, format: FileFormat.svg },

// By file extension (NO leading dot):
{ icon: 'name', extensions: ['ext1', 'ext2'], format: FileFormat.svg },

// By language ID:
{ icon: 'name', extensions: [], languages: [languages.langName], format: FileFormat.svg },

// With filename glob combinations:
{
  icon: 'name',
  extensions: ['.babelrc'],
  filenamesGlob: ['.babelrc', 'babel.config'],
  extensionsGlob: ['js', 'cjs', 'mjs', 'json'],
  filename: true,
  format: FileFormat.svg,
},
```

3. If using language IDs, add to `src/iconsManifest/languages.ts`:

```typescript
langName: { ids: 'vscode-language-id', knownExtensions: ['ext'] },
```

4. Optional: Create `icons/file_type_light_{name}.svg` if setting `light: true`
5. Run `npm test` to validate.

## Adding a Folder Icon

1. Create TWO SVG icons:
   - `icons/folder_type_{name}.svg` (collapsed)
   - `icons/folder_type_{name}_opened.svg` (expanded)
2. Add entry to `src/iconsManifest/supportedFolders.ts` in **alphabetical order**:

```typescript
{ icon: 'name', extensions: ['foldername', '.foldername'], format: FileFormat.svg },
```

3. Optional: Create light variants `folder_type_light_{name}.svg` + `folder_type_light_{name}_opened.svg`
4. Run `npm test` to validate.

## Adding a Language Association

In `src/iconsManifest/languages.ts`:

```typescript
langName: { ids: 'vscode-language-id', knownExtensions: ['ext'] },
```

- `ids`: VS Code language ID (string or string[])
- `knownExtensions`: File extensions for Zed editor support
- `knownFilenames`: Specific filenames for Zed editor support


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vscode-icons/vscode-icons](https://github.com/vscode-icons/vscode-icons) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
