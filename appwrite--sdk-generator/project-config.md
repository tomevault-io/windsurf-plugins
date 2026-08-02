---
trigger: always_on
description: This file provides guidance for AI agents (Claude Code and subagents) working in this repository.
---

# Agents Guide — SDK Generator

This file provides guidance for AI agents (Claude Code and subagents) working in this repository.

## Critical Rules for Agents

### Rule 1: Always regenerate after template edits

Templates are not used directly — they're rendered at generation time. The `examples/` folder is the ground truth for what the generator actually produces. Always regenerate after making template changes.

### Rule 2: New files require `getFiles()` registration

The generator does not auto-discover templates. Every output file must have an explicit entry in the `getFiles(): array` method of the corresponding Language class. Forgetting this means the file silently doesn't get generated.

### Rule 3: Check language inheritance before changing parent templates

| Parent | Children affected |
|--------|------------------|
| `Node` | `CLI`, `ReactNative` |
| `Dart` | `Flutter` |
| `Swift` | `Apple` |
| `Kotlin` | `Android` |

Modifying a parent's template or `getFiles()` affects all children. Regenerate and verify child SDKs too.

### Rule 4: `copy` scope = no Twig processing

Files with `'scope' => 'copy'` are copied verbatim — no variable substitution happens. If your new file needs template variables, use `'scope' => 'default'` (or `service`, `method`, etc.).

### Rule 5: Destination paths are also Twig templates

The `destination` string in each `getFiles()` entry supports Twig expressions and filters:
```php
'destination' => 'src/Services/{{ service.name | caseCamel }}.php',
```

### Rule 6: Never modify lock file templates directly

Lock file templates (`package-lock.json.twig`, `bun.lock.twig`) contain Twig expressions that get corrupted if you copy a raw lock file over them. Always use the update script:

```bash
./scripts/update-lockfiles.sh cli    # update CLI lock files only
./scripts/update-lockfiles.sh all    # update all TS-based SDK lock files
```

The script strips Twig expressions before running `npm install`/`bun install`, then restores them automatically. Never run `cp package-lock.json package-lock.json.twig` or edit these files by hand.

## Repository at a Glance

- **Purpose:** Generate Appwrite SDKs and tooling targets for 20+ languages/platforms from Swagger/OpenAPI specs using Twig templates
- **Language:** PHP (generator engine) + Twig (templates)
- **Entry point:** `example.php` — runs generation for all or a specific SDK
- **Output:** `examples/<lang>/` — checked-in generated SDK output for verification

```
src/SDK/Language/<Lang>.php   ← Language class: defines files, types, keywords
templates/<lang>/             ← Twig templates for that language
examples/<lang>/              ← Generated SDK output (checked in for verification)
example.php                   ← Entry point: regenerates all SDKs from specs
```

**Supported SDKs:** PHP, Web, Node, CLI, Ruby, Python, Dart, Flutter, React Native, Go, Swift, Apple, DotNet, Android, Kotlin, Unity, REST, GraphQL, Rust, Skills, CursorPlugin, ClaudePlugin, CodexPlugin

## Primary Workflows

### Modifying an Existing SDK Template

1. Edit template(s) in `templates/<lang>/`
2. Regenerate:
   ```bash
   php example.php <lang>
   ```
3. Diff `examples/<lang>/` to verify the output is correct
4. Run linters and refactor check:
   ```bash
   composer refactor:check
   composer lint-twig
   # or directly
   uvx djLint templates/ --lint
   ```

### Adding a New Template File to an Existing SDK

1. Create the `.twig` file in `templates/<lang>/`
2. **Register it in `src/SDK/Language/<Lang>.php` → `getFiles()` array** — this is mandatory:

```php
public function getFiles(): array
{
    return [
        // ...existing entries...
        [
            'scope'       => 'default',   // default|service|method|definition|requestModel|enum|copy|download
            'destination' => 'path/to/output.ext',
            'template'    => 'lang/path/to/template.twig',
        ],
    ];
}
```

**Scopes:**
- `default` — generated once per SDK (config files, README, main entry point)
- `service` — generated once per API service
- `method` — generated once per service×method combination
- `definition` — generated once per model/definition
- `requestModel` — generated once per request model
- `enum` — generated once per enum
- `copy` — static files copied as-is, no Twig processing
- `download` — generated once per SDK by downloading the URL in `template` to `destination`

3. Regenerate and verify

### Adding a New Language SDK

1. Create `src/SDK/Language/NewLang.php` (extend `Language` or a related language)
2. Implement: `getName()`, `getKeywords()`, `getIdentifierOverrides()`, `getFiles()`, `getTypeName()`, `getParamDefault()`, `getParamExample()`
3. Create `templates/newlang/` and add all Twig files
4. Register all template files in `getFiles()`
5. Add generation block to `example.php`
6. Generate: `php example.php newlang`
7. Inspect `examples/newlang/`

## File Reference Map

| What you want to change | Where to look |
|------------------------|---------------|
| Template for a language | `templates/<lang>/` |
| Which files get generated | `src/SDK/Language/<Lang>.php` → `getFiles()` |
| Type mappings for a language | `src/SDK/Language/<Lang>.php` → `getTypeName()` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [appwrite/sdk-generator](https://github.com/appwrite/sdk-generator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
