---
trigger: always_on
description: This document provides critical guidance for AI coding assistants working with Varying Vagrant Vagrants (VVV). **Read and follow these instructions carefully to avoid inappropriate changes.**
---

# VVV Development Environment Instructions for LLM Agents

This document provides critical guidance for AI coding assistants working with Varying Vagrant Vagrants (VVV). **Read and follow these instructions carefully to avoid inappropriate changes.**

## 🚨 Critical Boundaries - NEVER Cross These Lines

### DO NOT Modify These Core Files
- **`.gitignore`** - NEVER modify to allow committing `www/` content or other gitignored paths
- **`Vagrantfile`** - Core Vagrant configuration, do not modify without explicit permission
- **`/provision/`** scripts - Core provisioning logic (only modify for VVV improvements, not site work)
- **Git-tracked files** - Assume files tracked by git are VVV core files unless explicitly told otherwise

### Git Safety Rules - Follow These Strictly
1. **NEVER modify `.gitignore`** to allow committing `www/` content
2. **NEVER force commit** using `--no-verify`, `-f`, or similar flags
3. **ALWAYS check `git check-ignore <path>`** before committing files to verify they should be tracked
4. **If files in `www/` appear in git status** - they should almost certainly NOT be committed (except the defaults listed below)
5. **Ask before committing** if you're uncertain whether a file belongs in the VVV repository

## What is VVV?

VVV (Varying Vagrant Vagrants) is a **WordPress development environment** using Vagrant. It's infrastructure, not a project itself.

**Key Concept:** VVV is like Docker Compose for WordPress development. The `www/` folder is like Docker volumes - it contains local development sites that are **unique to each developer** and **should never be shared** via the VVV repository.

### Architecture
- **`www/` folder** - Contains local development WordPress sites (gitignored, except specific defaults)
- **`config/config.yml`** - User's site configuration (gitignored)
- **`provision/`** - Provisioning scripts that set up the Vagrant environment
- **`database/`** - MySQL data (gitignored)
- **`log/`** - Log files (gitignored)

## Working with WordPress Sites - The Correct Way

### Creating a New Site

When asked to create a new WordPress site:

**✓ CORRECT approach:**
1. Edit `/config/config.yml` (or create it from `config/default-config.yml` template)
2. Add a site configuration:
```yaml
sites:
  my-site-name:
    repo: https://github.com/Varying-Vagrant-Vagrants/custom-site-template.git
    php: 8.2
    hosts:
      - mysite.test
    custom:
      db_name: my_database
      site_title: "My Site"
      admin_user: admin
      admin_password: password
      install_plugins:
        - query-monitor
        - debug-bar
      wpconfig_constants:
        WP_DEBUG: true
        WP_DEBUG_LOG: true
```
3. Instruct user to run: `vagrant up --provision`
4. VVV will automatically create the site in `www/my-site-name/`

**✗ INCORRECT approach:**
- ❌ Creating directories directly in `www/my-site-name/`
- ❌ Downloading and copying WordPress files manually
- ❌ Modifying `.gitignore` to track the new site
- ❌ Committing site files to VVV repository
- ❌ Committing site files to site template repository

### Modifying an Existing Site

When asked to work on a WordPress site (themes, plugins, content):

**✓ CORRECT approach:**
1. Identify which site: "Which site in `www/` are you working on?"
2. Navigate to that site's directory: `www/site-name/public_html/`
3. Work within that site's directory structure:
   - Themes: `www/site-name/public_html/wp-content/themes/`
   - Plugins: `www/site-name/public_html/wp-content/plugins/`
   - Uploads: `www/site-name/public_html/wp-content/uploads/`
4. **DO NOT** offer to commit these changes to VVV repository
5. If user wants version control, suggest initializing git within their theme/plugin folder

**✗ INCORRECT approach:**
- ❌ Committing theme/plugin changes to VVV repository
- ❌ Creating pull requests with site-specific code
- ❌ Modifying VVV core to accommodate site-specific needs

### Site Configuration Options

The `custom:` section in `config/config.yml` supports extensive configuration:

```yaml
custom:
  # WordPress installation
  wp_version: latest              # Or specific version like "6.4.2"
  wp_type: single                 # Or "multisite"
  locale: en_US                   # WordPress language

  # Database
  db_name: my_database
  db_prefix: wp_

  # Admin user
  admin_user: admin
  admin_password: password
  admin_email: admin@local.test
  site_title: "My Site Title"

  # Plugins & themes
  delete_default_plugins: true    # Remove Akismet and Hello Dolly
  install_plugins:
    - query-monitor
    - debug-bar
    - https://github.com/user/plugin-repo/archive/main.zip
  install_themes:
    - twentytwentyfour

  # wp-config.php constants
  wpconfig_constants:
    WP_DEBUG: true
    WP_DEBUG_LOG: true
    WP_DEBUG_DISPLAY: false
    SCRIPT_DEBUG: true
```

## What CAN Be Committed to VVV

Only these types of changes are appropriate for VVV pull requests:
- 🐛 Bug fixes in VVV core provisioning scripts
- 📚 Documentation improvements
- ✨ New features for VVV itself (not sites)
- ⚙️ Default configuration updates
- 🔧 CI/CD improvements
- 🛠️ Improvements to VVV utilities and tools


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Varying-Vagrant-Vagrants/VVV](https://github.com/Varying-Vagrant-Vagrants/VVV) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
