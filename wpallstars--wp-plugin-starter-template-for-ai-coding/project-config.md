---
trigger: always_on
description: This guide helps AI assistants understand the project structure, workflows, and best practices for this repository.
---

# AI Assistant Guide for WordPress Plugin Development

This guide helps AI assistants understand the project structure, workflows, and best practices for this repository.

## IMPORTANT: Repository Context

This workspace may contain multiple repository folders.

Always focus ONLY on the current repository you're working in.

Avoid hallucinating functionality from other repositories in the workspace.

* **Current Repository**: wp-plugin-starter-template-for-ai-coding
* **Repository Purpose**: A starter template for WordPress plugins with AI-assisted development
* **Repository Scope**: All code changes and discussions should be limited to THIS repository only

## Project Overview

* **Plugin Name**: WordPress Plugin Starter Template
* **Plugin Slug**: wp-plugin-starter-template
* **Text Domain**: wp-plugin-starter-template
* **Namespace**: WPALLSTARS\PluginStarterTemplate
* **Version**: 0.1.15
* **Requires WordPress**: 5.0+
* **Requires PHP**: 7.4+
* **License**: GPL-2.0+

## Repository Structure

* **wp-plugin-starter-template.php**: Main plugin file with plugin headers
* **includes/**: Core plugin functionality
  * **plugin.php**: Main plugin class that initializes everything
  * **core.php**: Core functionality class
  * **updater.php**: Update mechanism for multiple sources
* **admin/**: Admin-specific functionality
  * **lib/**: Admin classes
  * **css/**: Admin stylesheets
  * **js/**: Admin JavaScript files
* **languages/**: Translation files
* **.github/workflows/**: GitHub Actions workflows
* **.agents/**: Documentation for AI assistants
* **.wiki/**: Wiki documentation templates

## Coding Standards

This project follows the [WordPress Coding Standards](https://developer.wordpress.org/coding-standards/):

* Use 4 spaces for indentation, not tabs (this is a project-specific override of WordPress standards)
* Follow WordPress naming conventions:
  * Class names: `Class_Name`
  * Function names: `function_name`
  * Variable names: `$variable_name`
* Use proper DocBlocks for all classes, methods, and functions
* Ensure all user-facing strings are translatable
* Validate and sanitize all inputs
* Escape all outputs
* Use asterisks (*) for bullet points in all Markdown files, not hyphens (-)
* Add periods to the end of all inline comments

### Code Quality Tools

This project uses several automated code quality tools to ensure high standards:

1. **PHP_CodeSniffer (PHPCS)**: Checks PHP code against the WordPress Coding Standards
   ```bash
   composer run phpcs
   ```

2. **PHP Code Beautifier and Fixer (PHPCBF)**: Automatically fixes some coding standard violations
   ```bash
   composer run phpcbf
   ```

3. **ESLint**: Checks JavaScript code against the WordPress Coding Standards
   ```bash
   npm run lint:js
   ```

4. **Stylelint**: Checks CSS code against the WordPress Coding Standards
   ```bash
   npm run lint:css
   ```

5. **Continuous Integration Tools**: The project integrates with several code quality tools:
   * **CodeRabbit**: AI-powered code review tool
   * **CodeFactor**: Continuous code quality monitoring
   * **Codacy**: Code quality and static analysis
   * **SonarCloud**: Code quality and security analysis

Always run PHPCS and PHPCBF locally before committing code to ensure it meets the project's coding standards.

## AI Tool Discipline

Recurring tool failures detected from real sessions in this repository. Follow these rules to avoid them.

For detailed guidance, see **@.agents/ai-tool-best-practices.md**.

### Read Before Edit (critical — 71 `edit:not_read_first` failures)

* ALWAYS use the Read tool on a file before using Edit or Write on it.
* Never assume file content — always read first, even for small changes.
* If you already read the file earlier in the session, re-read it if any other tool has modified it since.

### Re-Read After Edit (32 `edit:edit_stale_read` failures)

* After editing a file, always re-read it before making additional edits to the same file.
* In-memory content becomes stale immediately after any modification.
* Pattern: Read → Edit → Re-read → Edit (never Read → Edit → Edit).

### Verify File Paths Before Reading (23 `read:file_not_found` failures)

* Before reading a file, verify it exists: `git ls-files '<pattern>'` for tracked files.
* Key tracked paths in this repo: `wp-plugin-starter-template.php`, `includes/`, `admin/`, `languages/`, `.agents/`.
* If a path is uncertain, use `git ls-files | grep <keyword>` to find the correct path.
* Never assume a file exists — verify first.

### Bash Command Hygiene (66 `bash:other` failures)

* Verify the working directory before running commands — use `pwd` if uncertain.
* Check prerequisites exist before using them (e.g., `composer`, `npm`, `php`).
* Use `|| true` when a command failure is acceptable in a pipeline.
* For commands that may fail, check exit codes explicitly.
* Common root causes: wrong working directory, missing tool, incorrect path, shell syntax error.

## Common Tasks

For detailed instructions on releases, features, bugs, and testing, see **@.agents/release-process.md**.

For local testing with WordPress Playground, LocalWP, and wp-env, see **@.agents/local-testing-guide.md**.

## Avoiding Cross-Repository Confusion


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wpallstars/wp-plugin-starter-template-for-ai-coding](https://github.com/wpallstars/wp-plugin-starter-template-for-ai-coding) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
