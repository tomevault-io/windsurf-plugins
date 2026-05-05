---
trigger: always_on
description: This is a WordPress plugin
---

# Copilot Instructions

## Repository Overview

This is a WordPress plugin

- **Repository Type:** WordPress Plugin
- **Primary Language:** PHP, JavaScript (React/JSX)
- **Frameworks:** WordPress, WordPress Block Editor (Gutenberg), @wordpress/scripts
- **Target Runtime:** WordPress 6.8+, PHP 8.1+, Node.js 20+

## Critical Build Instructions

### Environment Requirements

- **Node.js:** greater than v20.19.5
- **npm:** greater than v10.8.2
- **PHP:** 8.1 or higher (plugin requires PHP 8.1+)
- **Composer:** 2.8.12 or higher

### Dependency Versions

#### npm Packages
- **@wordpress/scripts:** 30.26.0 or higher

#### Composer Packages
- **wp-coding-standards/wpcs:** 3.0 or higher

### Dependency Installation

**ALWAYS install dependencies in this exact order before any build operations:**

1. **npm dependencies (REQUIRED FIRST):**
   ```bash
   npm install
   ```
   - Takes approximately 60 seconds on first install
   - May show deprecation warnings (these are non-critical)
   - May show 2 moderate severity vulnerabilities (these are from dev dependencies and are non-critical)
   - Creates `node_modules/` directory (ignored by git)

2. **Composer dependencies (for PHP linting only):**
   ```bash
   composer install --no-interaction
   ```
   - Takes approximately 30-60 seconds
   - May prompt for GitHub OAuth token if run interactively; use `--no-interaction` flag to avoid this
   - Falls back to cloning from git cache if GitHub API rate limits are hit
   - Creates `vendor/` directory (ignored by git)
   - Installs WordPress Coding Standards (WPCS) for PHP linting

### Build Process

**ALWAYS run build after making JavaScript/CSS changes:**

```bash
npm run build
```
- Takes approximately 1-2 seconds
- Uses webpack via @wordpress/scripts
- Generates files in `build/` directory (ignored by git, but required for plugin to function)
- Creates `build/blocks-manifest.php` (automatically generated, do NOT edit manually)
- Creates minified JavaScript, CSS, and asset dependency files in `build/[your-block-name]/`
- Build output includes RTL CSS variants automatically

**For development with hot reload:**
```bash
npm run start
```
- Watches for file changes and rebuilds automatically
- Generates unminified source maps for debugging
- Use Ctrl+C to stop the watch process

**To clean and rebuild:**
```bash
rm -rf build/
npm run build
```

### Linting and Code Quality

**JavaScript/JSX Linting:**
```bash
npm run lint:js
```
- Uses ESLint with WordPress coding standards
- Checks all `.js` files in `src/`
- Must pass with no errors before committing
- Some warnings are acceptable

**CSS/SCSS Linting:**
```bash
npm run lint:css
```
- Uses stylelint with WordPress standards
- Checks all `.scss` files in `src/`
- Must pass with no errors before committing

**Auto-formatting:**
```bash
npm run format
```
- Uses Prettier to auto-format JavaScript, JSON, CSS/SCSS
- **ALWAYS run this before linting if you get Prettier errors**
- Automatically fixes most lint issues related to formatting
- Safe to run on all files

**PHP Linting:**
```bash
./vendor/bin/phpcs [your-plugin-file.php]
```
- Uses PHP_CodeSniffer with WordPress Coding Standards
- Available coding standards: WordPress, WordPress-Core, WordPress-Docs, WordPress-Extra
- The main plugin file may have known PHPCS warnings (tabs vs spaces, line length) - these may be acceptable per project style
- Auto-fix many PHP issues with: `./vendor/bin/phpcbf [your-plugin-file.php]`

**NOTE:** This project uses **tabs for indentation** (not spaces) per WordPress coding standards, as specified in `.editorconfig`. The PHPCS errors about "spaces must be used" are using the wrong standard and can be ignored.

### Plugin Distribution

**To create a distributable ZIP file:**
```bash
npm run plugin-zip
```
- Creates `[your-plugin-name].zip` in the repository root (ignored by git)
- Includes only necessary files: `[your-plugin-file.php]`, `build/` directory contents
- Excludes: `src/`, `node_modules/`, `vendor/`, development files
- Takes approximately 1-2 seconds

## Project Architecture

### Directory Structure

```
[your-plugin-name]/
├── .editorconfig              # Editor configuration (tabs, not spaces)
├── .gitignore                 # Excludes build/, node_modules/, vendor/, *.zip
├── composer.json              # PHP dependencies (WordPress Coding Standards)
├── package.json               # npm dependencies and scripts
├── [your-plugin-file.php]     # Main plugin file - registers blocks
└── src/
    └── [your-block-name]/     # Block source files
        ├── block.json         # Block metadata (schema, scripts, styles)
        ├── index.js           # Block registration entry point
        ├── edit.js            # Block editor component (React)
        ├── save.js            # Block save/render component (React)
        ├── view.js            # Front-end JavaScript
        ├── editor.scss        # Editor-only styles
        └── style.scss         # Front-end and editor styles
```

### Build Output Structure

```
build/                         # Generated by npm run build (DO NOT EDIT)
├── blocks-manifest.php        # Auto-generated block metadata
└── [your-block-name]/
    ├── block.json            # Copied from src
    ├── index.js              # Minified editor script

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jonathanbossenger/wordpress-plugin-copilot-instructions](https://github.com/jonathanbossenger/wordpress-plugin-copilot-instructions) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
