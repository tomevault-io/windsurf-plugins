---
trigger: always_on
description: **Project:** Vibe Code Deploy WordPress Plugin
---

# Cursor Development Rules - Vibe Code Deploy Plugin

**Project:** Vibe Code Deploy WordPress Plugin  
**Purpose:** Enforce WordPress plugin development best practices and coding standards  
**Last Updated:** 2025

---

## WordPress Plugin Development Standards

### Required Standards

All code must follow **WordPress Plugin Development Best Practices**:

- **Reference**: See `docs/WORDPRESS_PLUGIN_BEST_PRACTICES.md` for comprehensive guidelines
- **WordPress Plugin Handbook**: https://developer.wordpress.org/plugins/
- **WordPress Coding Standards**: https://developer.wordpress.org/coding-standards/

### Plugin Lifecycle Hooks

**REQUIRED**: All plugins must implement proper activation, deactivation, and uninstall hooks.

- **Activation Hook**: Set default options, flush rewrite rules, schedule cron jobs
- **Deactivation Hook**: Clear scheduled events, flush rewrite rules
- **Uninstall Hook**: Delete options, transients, clean up database (preserve user data)

**Reference**: See `docs/WORDPRESS_PLUGIN_BEST_PRACTICES.md#plugin-lifecycle-hooks`

### Security Requirements

**ALWAYS**:
- Check `ABSPATH` at the top of all PHP files
- Use `current_user_can( 'manage_options' )` for admin pages
- Use `check_admin_referer()` or `wp_verify_nonce()` for form submissions
- Sanitize all input with WordPress functions (`sanitize_text_field()`, `sanitize_email()`, etc.)
- Escape all output with WordPress functions (`esc_html()`, `esc_attr()`, `esc_url()`)
- Use prepared statements for database queries

**Reference**: See `docs/WORDPRESS_PLUGIN_BEST_PRACTICES.md#security-best-practices`

### Code Organization

**File Structure**:
```
plugin-name/
├── plugin-name.php          # Main plugin file
├── uninstall.php            # Uninstall handler
├── includes/
│   ├── Bootstrap.php       # Plugin initialization
│   ├── Settings.php        # Settings management
│   ├── Services/           # Service classes
│   └── Admin/              # Admin pages
├── assets/
│   ├── css/
│   └── js/
└── tests/                  # Unit tests
```

**Namespace**: Use `VibeCode\Deploy` for core, `VibeCode\Deploy\Services` for services, `VibeCode\Deploy\Admin` for admin pages.

### PHP Standards

**WordPress PHP Coding Standards**:
- **Indentation**: Use tabs (not spaces)
- **Brace Style**: Opening brace on same line
- **Naming**: Use `snake_case` for functions, `PascalCase` for classes
- **Type Declarations**: Use type hints for parameters and return types (PHP 8.0+)
- **Visibility**: Always declare visibility (`public`, `private`, `protected`)
- **Final Classes**: Use `final` for service classes that shouldn't be extended

**Reference**: See `docs/WORDPRESS_PLUGIN_BEST_PRACTICES.md#wordpress-coding-standards`

### Internationalization (i18n)

**REQUIRED**: All user-facing strings must be translatable.

- Use text domain: `vibecode-deploy`
- Use WordPress translation functions: `__()`, `_e()`, `esc_html__()`, `esc_html_e()`
- Always escape translated output

**Reference**: See `docs/WORDPRESS_PLUGIN_BEST_PRACTICES.md#internationalization-i18n`

### Database Management

**REQUIRED**: Track database schema versions and handle upgrades.

- Use `dbDelta()` for table creation
- Track schema version in options
- Handle upgrades in activation hook or upgrade routine

**Reference**: See `docs/WORDPRESS_PLUGIN_BEST_PRACTICES.md#database-management`

### Version Management

**🚨 CRITICAL RULES:**

1. **Auto-increment on build** - The build script (`scripts/build-plugin-zip.sh`) automatically increments the patch version (third number) by 1 on each build
   - Example: `0.1.1` → `0.1.2` → `0.1.3`
   - The script updates both the `Version:` header and `VIBECODE_DEPLOY_PLUGIN_VERSION` constant
2. **Do NOT manually edit version** - Never manually change version numbers in `vibecode-deploy.php`; the build script handles this automatically
3. **Version format**: Semantic versioning (`MAJOR.MINOR.PATCH`, e.g., `0.1.2`)
4. **Why**: Version increments ensure browser cache busting for CSS/JS assets and proper upgrade detection

**Example**:
```php
// vibecode-deploy.php (auto-updated by build script)
 * Version: 0.1.2
define( 'VIBECODE_DEPLOY_PLUGIN_VERSION', '0.1.2' );
```

**Manual version edits (if needed):**
- Only edit version manually if you need to increment MAJOR or MINOR (not PATCH)
- PATCH version is always auto-incremented by the build script
- If you manually change MAJOR or MINOR, the next build will increment PATCH from your manual value

### Build Process

**🚨 CRITICAL RULES:**

1. **ALWAYS build plugin zip after code changes** - After making ANY changes to plugin code, you MUST build the plugin zip:
   ```bash
   cd /Users/tlowing/CascadeProjects/windsurf-project/VibeCodeCircle
   ./scripts/build-plugin-zip.sh
   ```
   - The zip file is required for WordPress installation
   - Build script is at: `scripts/build-plugin-zip.sh`
   - Output goes to: `dist/vibecode-deploy-{version}.zip`
   - **Never commit without building the zip first**

2. **Only versioned zip is created** - The build script creates only `dist/vibecode-deploy-{version}.zip`, NOT an unversioned `vibecode-deploy.zip`
   - Example output: `dist/vibecode-deploy-0.1.2.zip`
   - The unversioned zip is redundant and not created

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/TrevorLowing) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
