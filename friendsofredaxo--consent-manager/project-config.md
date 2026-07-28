---
trigger: always_on
description: This is the **REDAXO Consent Manager AddOn** - a comprehensive GDPR/DSGVO-compliant cookie consent management system for REDAXO CMS. The project provides advanced consent management with inline-consent system, accessibility features, and extensive theming capabilities.
---

# REDAXO Consent Manager - Copilot Instructions

## Project Overview

This is the **REDAXO Consent Manager AddOn** - a comprehensive GDPR/DSGVO-compliant cookie consent management system for REDAXO CMS. The project provides advanced consent management with inline-consent system, accessibility features, and extensive theming capabilities.

**Repository:** FriendsOfREDAXO/consent_manager  
**Main Language:** PHP (REDAXO CMS AddOn)  
**Frontend:** JavaScript, SCSS/CSS  
**Database:** MySQL/MariaDB  

## Architecture & Key Components

### Core Structure
```
├── boot.php              # AddOn bootstrap file
├── package.yml           # AddOn configuration
├── lib/                  # Core PHP classes
│   ├── consent_manager_frontend.php    # Frontend controller
│   ├── consent_manager_theme.php       # Theme system
│   ├── consent_manager_inline.php      # Inline consent system
│   └── consent_manager_*.php           # Various utility classes
├── pages/                # Backend administration pages
│   ├── theme.php         # Theme selection and preview
│   ├── theme_editor.php  # A11y theme editor with color picker
│   └── *.php            # Domain, service, text management
├── fragments/            # Template fragments
│   ├── consent_manager_box.php         # Main consent dialog
│   └── consent_inline_placeholder.php  # Inline consent placeholder
├── assets/              # Frontend assets
│   ├── *.js             # JavaScript files (main, polyfills, debug)
│   └── *.css            # Compiled CSS themes
├── scss/                # SCSS source files
│   ├── consent_manager_frontend*.scss  # Theme sources
│   └── consent_manager_frontend_a11y*.scss # Accessibility themes
└── setup/               # Setup configurations
    └── *.json           # Default setup data
```

### Database Schema
- `rex_consent_manager_domain` - Domain configurations
- `rex_consent_manager_domain_service` - Service assignments  
- `rex_consent_manager_service` - Service definitions
- `rex_consent_manager_text` - Multilingual texts
- `rex_consent_manager_log` - Consent logging

## Development Guidelines

### Code Standards
- **PHP:** Follow REDAXO conventions, use `rex_` prefixes for classes
- **Naming:** Use snake_case for files, camelCase for methods
- **Security:** Always use `rex_escape()` for output, `rex_request()` for input
- **i18n:** Use `$addon->i18n('key')` for all translatable strings
- **Database:** Use `rex_sql` class, never direct SQL queries

### Commit Messages
Use **Conventional Commits** format (configured in `.gitmessage`):
```
<type>(<scope>): <subject>

Types: feat, fix, docs, style, refactor, test, chore, perf
Scopes: theme, inline, a11y, frontend, backend, config
```

### CSS/SCSS Guidelines
- **Themes:** Create new themes in `scss/consent_manager_frontend_theme_*.scss`
- **Variables:** Use SCSS variables for colors, spacing, fonts
- **Accessibility:** Follow WCAG 2.1 AA guidelines (4.5:1 contrast ratio)
- **BEM-style:** Use `.consent_manager-` prefix for all classes

### JavaScript Guidelines  
- **ES5 Compatible:** Support older browsers
- **No Dependencies:** Vanilla JS only (except js.cookie.min.js)
- **Event Delegation:** Use `data-` attributes instead of `onclick`
- **Debugging:** Use `consent_debug.js` for development logging

## Build & Development

### Theme Development
```bash
# Test theme preview (automatic compilation)
/redaxo/index.php?page=consent_manager/theme&preview=theme_name

# SCSS is automatically compiled when theme is selected in backend
# No manual sass-watch or compilation needed
```

### Setup Files and UIDs
**Important:** New text UIDs must be added to ALL setup files:
- `setup/minimal_setup.json` - Basic setup with essential services
- `setup/default_setup.json` - Standard setup with 25+ preconfigured services  
- `setup/business_setup.json` - Business-oriented services
- `setup/contribution_template.json` - Template for community contributions

**When adding new text UIDs:**
1. Add new UID to all 4 setup files
2. Number `id` sequentially per file
3. Use same `uid` and `text` in all files
4. Follow JSON structure (see existing entries)

### Git Workflow & Commits
```bash
# Setup development environment (one-time)
make setup-git

# Commit messages: ALWAYS use Conventional Commits format
git commit  # (without -m) opens template with examples

# Good commit examples for this project:
feat(inline): Add new inline consent functionality
fix(theme): Fix SCSS compilation for A11y themes  
docs(setup): Add new text UID to all setup files
style(a11y): Improve contrast values for WCAG 2.1 AA
refactor(frontend): Extract event delegation to separate function
chore(i18n): Update German translations

# Pull Requests: 
# - Description must explain WHAT and WHY
# - For setup changes: mention all affected files
# - For theme changes: document accessibility tests
# - Take Copilot reviews seriously and fix them
```

### Testing
- **Frontend:** Test in multiple browsers, check accessibility
- **Backend:** Test in REDAXO backend with different user permissions  

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FriendsOfREDAXO/consent_manager](https://github.com/FriendsOfREDAXO/consent_manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
