---
trigger: always_on
description: This file provides guidance to an AI assistant when working with the code in this repository.
---

# AGENTS.md

This file provides guidance to an AI assistant when working with the code in this repository.

## Project Overview

This is the **OpenSSF Best Practices Badge** project (formerly CII Best Practices Badge) - a Rails web application that helps FLOSS projects self-certify that they meet security and quality best practices. The application provides a badging system with three "metal" levels (passing, silver, and gold) and three "baseline" levels (1, 2, and 3).

This software is open source software. The production site running this code is extremely busy and always under attack. High performance and strong security are required. Minimize creating new objects after initialization, as unchecked memory growth leads to crashes.

Long-term maintainability is critical. Minimize the amount of code by preferring to reuse existing infrastructure and refactoring common code into methods.

When you have difficulty fixing a bug or making a change, determine the root cause and find ways to systemically prevent similar problems in the future. For example:

* Add a test to detect similar problems
* Improve bug feedback mechanisms (better error messages, fail-fast checks)
* Add documentation near the construct in use

However, documentation is often unnoticed and too much can obscure what's important. Prefer automated prevention (tests, type checks, linters) over documentation when possible.

**Key URLs:**

- Production: https://www.bestpractices.dev/
- GitHub: https://github.com/ossf/best-practices-badge

## Common Development Commands

### Testing

- `rake test:optimized` - Run all tests (unit and system). Takes a long time.
- `rails test test/integration/project_list_test.rb` - Run specified test file
- `rails test test/features/can_access_home_test.rb:4` - Run a test at line 4 of the specified test file.

To see test names, set environment variable SLOW=true e.g.,
`SLOW=true rake test:optimized`.

We require 100% statement coverage. We avoid using :nocov: because Ruby
only does dynamic type-checking. Lines of code that are *never*
tested have an increased risk of causing production failures.

### Code Quality & Linting

- `rake default` - Run local CI/CD pipeline (linting, tests, etc.).

   Don't just use `rake` without arguments, Claude's user permission
   system can't permit that without permitting all rake commands.
   Some additional checks (e.g., Brakeman) run on GitHub's CI/CD pipeline.

- `rake rubocop` - Ruby style checker
- `rake rails_best_practices` - Rails-specific best practices source checker
- `rake markdownlint` - Markdown linting
- `rake eslint` - JavaScript linting
- `rake whitespace_check` - Check for trailing whitespace
- `rake yaml_syntax_check` - YAML syntax validation
- `rake license_okay` - License compliance check
- `rake bundle_audit` - Security audit of gems

For specific files:

- `mdl FILENAME.md` - Markdownlint only the file `FILENAME.md`
- `rubocop FILENAME.rb` - Run rubocop on just `FILENAME.rb`

**Markdown Helper Script**:

After creating or editing markdown files, youcan use the markdown fixer script to automatically fix common markdownlint errors:

```bash
# Fix markdown file in-place (most common usage):
script/fix_markdown.rb docs/MYFILE.md

# See what would be fixed without changing the file:
script/fix_markdown.rb --dry-run docs/MYFILE.md

# After fixing, verify with markdownlint:
mdl docs/MYFILE.md
```

Don't run `rails_best_practices` to analyze individual files.
The `rails_best_practices` program needs the full context of all files
to do its best.

**IMPORTANT**: You may ONLY use `rubocop -a` (safe autocorrect)
for RuboCop corrections.

**NEVER use `rubocop -A` (unsafe autocorrect)** -
it frequently introduces subtle bugs by making assumptions
about code context that are incorrect.

### Workflow

After making significant changes to source code, ALWAYS run linters, e.g.,
rubocop for Ruby files and mdl for markdown files.

Also run `rake whitespace_check` to catch trailing whitespace, and fix them.

### Development Environment Shortcut

As a convenience, in the development environment you don't need to use
`bundle exec` prefixes for ruby commands (though you may use them).
They *are* necessary in cases, such as rake tasks, that might be
used in the CI or production environments.

### Security Analysis

GitHub runs Brakeman for static security analysis via
`.github/workflows/brakeman.yml` (gating the main branch), and CodeQL where
configured; this is not done on the local system. Brakeman's accepted
findings are recorded in `config/brakeman.ignore`.

### Development Server

- `rails s` - Start development server (http://localhost:3000)
- `rails console` - Start Rails console for debugging

## Architecture & Key Concepts

### Core Models

- **Project** - Central model representing a FLOSS project seeking certification
  - Has three badge levels: passing, silver, gold (plus `in_progress`)
  - Uses PostgreSQL full-text search via `pg_search` gem
  - Stores criteria answers and justifications
  - Heavy security validation and input sanitization

- **User** - Project owners and contributors
  - Email addresses are encrypted in database using `attr_encrypted`
  - Uses OAuth (GitHub) and local authentication

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [coreinfrastructure/best-practices-badge](https://github.com/coreinfrastructure/best-practices-badge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
