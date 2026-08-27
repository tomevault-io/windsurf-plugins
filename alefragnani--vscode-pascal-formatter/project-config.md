---
trigger: always_on
description: Use when reviewing pull requests that add or update localization files (package.nls.*.json, bundle.l10n.*.json, walkthrough .nls.*.md files) for VS Code extensions. Validates key completeness, JSON syntax, placeholder preservation, and file structure.
---


# Localization Pull Request Review Instructions

These instructions guide the Copilot Code Review agent when reviewing pull requests that add or update localization files for the VS Code extension.

## Overview

This extension supports multiple localization approaches:
1. **Package contributions** (`package.nls.*.json`) - for settings, commands, menus, views, and walkthrough metadata
2. **Runtime messages** (`l10n/bundle.l10n.*.json`) - for messages displayed in extension code

## Language Code Format

All language identifiers must follow the BCP 47 format and match VS Code's supported languages:
- Use lowercase for language codes: `es`, `fr`, `pt-br`, `zh-cn`, `zh-tw`
- Use hyphen separator for regional variants: `pt-br` (not `pt_br` or `pt-BR`)
- Verify the language code is supported by VS Code

## File Naming Conventions

### Package Localization Files
- **Pattern**: `package.nls.{LANGID}.json`
- **Location**: Root directory
- **Example**: `package.nls.es.json`, `package.nls.pt-br.json`

### Runtime Bundle Localization Files
- **Pattern**: `bundle.l10n.{LANGID}.json`
- **Location**: `l10n/` directory
- **Example**: `bundle.l10n.es.json`, `bundle.l10n.pt-br.json`

## Critical Validation Checks

### 1. Key Completeness and Consistency

**For `package.nls.*.json` files:**
- ✅ All keys from `package.nls.json` must be present in the localized file
- ✅ No extra keys should exist that aren't in the base file
- ✅ Key names must match exactly (case-sensitive)
- ❌ Missing keys mean incomplete localization
- ❌ Extra keys indicate misalignment with base file

**For `bundle.l10n.*.json` files:**
- ✅ All keys from `l10n/bundle.l10n.json` must be present
- ✅ Check for proper translation of variable placeholders (e.g., `{0}`, `{1}`)
- ✅ Preserve format specifiers and escape sequences

### 2. JSON Format and Syntax

- ✅ Valid JSON syntax (no trailing commas, proper quotes, balanced braces)
- ✅ UTF-8 encoding without BOM
- ✅ Consistent indentation (4 spaces per the project's style)
- ✅ Keys and values properly quoted with double quotes
- ✅ Proper escaping of special characters (`\n`, `\"`, `\\`)
- ❌ No single quotes for strings
- ❌ No comments in JSON files

### 3. Translation Quality Indicators

While you cannot verify translation accuracy, watch for:
- ⚠️ Values that are identical to the English version (may indicate incomplete translation)
- ⚠️ Missing diacritics or special characters expected in the target language
- ⚠️ Broken variable placeholders: `{0}`, `{1}` must be preserved
- ⚠️ Broken markdown links or command references in walkthrough files
- ⚠️ Inconsistent use of terms across the same file

### 4. Structural Integrity

**Package files:**
- ✅ Structure matches `package.nls.json` exactly
- ✅ Nested properties preserved (e.g., `"extId.configuration.someConfig.description"`)
- ✅ Array elements and object structures maintained

**Bundle files:**
- ✅ Multi-line strings preserve `\n` characters
- ✅ Format placeholders maintained: `{0}`, `{1}`, etc.
- ✅ Contextual consistency with surrounding text

**Walkthrough files:**
- ✅ Markdown syntax valid
- ✅ Command links intact: `(command:extId.someCommand)`
- ✅ Image references preserved
- ✅ Code blocks and formatting maintained

### 5. Full Localization Pack Validation

When a new language is added, ensure that all three types of localization files are included:
- `package.nls.{LANGID}.json`
- `l10n/bundle.l10n.{LANGID}.json`

## Common Issues to Flag

### High Priority Issues ❌

1. **Missing keys** - Incomplete localization
   ```
   Missing keys: ["extId.commands.someCommand.title"]
   ```

2. **Invalid JSON syntax** - File cannot be parsed
   ```
   Syntax error at line 45: Unexpected token ,
   ```

3. **Broken placeholders** - Runtime errors possible
   ```
   English: "Some error occured: {0}"
   Localized: "Algum erro ocorreu" // Missing {0}
   ```

4. **Wrong file location** - Files won't be loaded
   ```
   ❌ src/l10n/bundle.l10n.pt-br.json  (wrong location)
   ✅ l10n/bundle.l10n.pt-br.json      (correct location)
   ```

5. **Incomplete localization pack** - Missing files for a new language
   ```
   ❌ Missing files for language 'pt-br':
   - `package.nls.pt-br.json`
   - `l10n/bundle.l10n.pt-br.json`
   ```

### Medium Priority Issues ⚠️

1. **Extra keys** - May indicate stale translations
   ```
   Extra keys not in base file: ["extId.commands.oldCommand.title"]
   ```

2. **Untranslated values** - Values identical to English
   ```
   "extId.commands.newCommand.title": "New Command"  // Appears untranslated
   ```

3. **Inconsistent formatting** - Reduce diff noise
   ```
   // Inconsistent indentation or line endings
   ```

### Low Priority Issues ℹ️

1. **Missing newline at end of file**
2. **Inconsistent quote escaping** (if functionally equivalent)
3. **Whitespace inconsistencies** (if not affecting output)

## Review Checklist

When reviewing a localization PR, verify:

- [ ] Language code is correctly formatted and consistent
- [ ] Files are placed in the correct directories
- [ ] File naming follows conventions exactly

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alefragnani/vscode-pascal-formatter](https://github.com/alefragnani/vscode-pascal-formatter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
