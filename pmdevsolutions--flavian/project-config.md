---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **Claude Code-integrated WordPress development template** providing a clean `wp-content` directory structure with WordPress-specific development tools and scripts.

The template is designed for:
- WordPress FSE (Full Site Editing) block theme development
- Custom WordPress plugin development
- WordPress security, performance, and accessibility auditing
- Integration with Claude Code for WordPress development workflows

## ⚠️ CRITICAL: File Location Requirements

**This project uses ROOT-LEVEL WordPress folders, NOT `wp-content/` subfolders:**

```
project-root/
├── themes/          ← Theme files go HERE (NOT wp-content/themes/)
├── plugins/         ← Plugin files go HERE (NOT wp-content/plugins/)
├── mu-plugins/      ← Must-use plugins go HERE (NOT wp-content/mu-plugins/)
└── .claude/         ← Claude Code configuration
```

**Why root-level?**
- Cleaner development structure
- Easier version control (no nested wp-content)
- Testing copies files to WordPress `wp-content/` for deployment
- Development and testing environments separated

**NEVER create files in `wp-content/themes/` or `wp-content/plugins/` during development.**

When deploying to WordPress, files are copied from root folders to `wp-content/` by deployment scripts.

## WordPress Development Scripts

This template includes WordPress-specific automation scripts:

### Security and Quality Tools
```bash
# Set up PHP CodeSniffer with WordPress standards
./scripts/wordpress/setup-phpcs.sh

# Check WordPress coding standards
./scripts/wordpress/check-coding-standards.sh

# Run security scan
./scripts/wordpress/security-scan.sh

# Check performance
./scripts/wordpress/check-performance.sh
```

## Development Commands

### WordPress Setup
```bash
# Download WordPress core files
wp core download

# Create wp-config.php
wp config create --dbname=dbname --dbuser=root --dbpass=password --dbhost=localhost

# Install WordPress
wp core install --url=example.com --title="Site Title" --admin_user=admin --admin_password=password --admin_email=admin@example.com

# Update WordPress core
wp core update

# Update all plugins
wp plugin update --all

# Update all themes
wp theme update --all
```

### Theme Development
```bash
# Activate a theme
wp theme activate theme-name

# Generate theme starter files
wp scaffold _s theme-name --theme_name="Theme Name" --author="Author Name"

# Check theme for errors
wp theme verify theme-name
```

### Plugin Development
```bash
# Activate a plugin
wp plugin activate plugin-name

# Deactivate a plugin
wp plugin deactivate plugin-name

# Generate plugin starter files
wp scaffold plugin plugin-name --plugin_name="Plugin Name"

# Run plugin tests (if PHPUnit is configured)
phpunit
```

### Database Operations
```bash
# Export database
wp db export backup.sql

# Import database
wp db import backup.sql

# Search and replace in database
wp search-replace 'old-domain.com' 'new-domain.com'

# Reset database
wp db reset
```

### Development Server
```bash
# Using PHP built-in server
php -S localhost:8000

# Using wp-cli server
wp server --host=localhost --port=8080

# Using Local by Flywheel, XAMPP, MAMP, or Docker
# Configure according to your local environment
```

## WordPress File Structure

When developing, follow these conventions:

### Theme Structure (Development - Root Level)
```
themes/theme-name/              ← ROOT-LEVEL (not wp-content/themes/)
├── style.css                   # Theme information and main styles
├── functions.php               # Theme functions and hooks
├── index.php                   # Main template file (classic themes)
├── theme.json                  # FSE theme configuration (REQUIRED for FSE)
├── templates/                  # FSE block templates
│   ├── index.html             # Main template
│   ├── front-page.html        # Homepage template
│   ├── single.html            # Single post template
│   ├── page.html              # Page template
│   └── 404.html               # 404 error template
├── parts/                      # FSE template parts
│   ├── header.html            # Header template part
│   ├── footer.html            # Footer template part
│   └── sidebar.html           # Sidebar template part
├── patterns/                   # Block patterns
├── assets/
│   ├── css/                   # Additional stylesheets
│   ├── js/                    # JavaScript files
│   └── images/                # Theme images
└── inc/                        # PHP includes

# Classic theme alternatives (if not using FSE):
# ├── header.php, footer.php, sidebar.php
# ├── single.php, page.php, archive.php
# └── template-parts/
```

### Plugin Structure (Development - Root Level)
```
plugins/plugin-name/            ← ROOT-LEVEL (not wp-content/plugins/)
├── plugin-name.php            # Main plugin file with header
├── includes/                  # PHP includes
├── admin/                     # Admin-specific functionality
├── public/                    # Public-facing functionality
├── assets/
│   ├── css/
│   ├── js/
│   └── images/
└── languages/                 # Translation files
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PMDevSolutions/Flavian](https://github.com/PMDevSolutions/Flavian) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
