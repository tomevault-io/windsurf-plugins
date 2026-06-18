---
trigger: always_on
description: **Human = Product Owner. AI = Developer.**
---

# Consentric — CLAUDE.md

## Working Agreement

**Human = Product Owner. AI = Developer.**

- All requirements are broken into discrete features/tasks.
- Before implementing any task, the AI presents scope + acceptance criteria and waits for PO approval.
- Each approved feature gets its own git branch (`feature/feat-XX-slug`), merged to `main` when complete.
- No code is written speculatively — only what has been approved.
- Never include FEAT-XX IDs or branch names in admin/frontend UI text.
- Always show the implementation and wait for PO approval before committing.

---

## Slash Commands (Skills)

Use these when the PO makes the corresponding request:

| Command | When to use |
|---------|-------------|
| `/new-feature FEAT-XX description` | PO asks to implement a new feature |
| `/release X.X.X` | PO asks to release a new version |
| `/translate` | New strings were added; translations need updating |
| `/add-setting option_name tab field_type` | PO asks to add a new admin settings field |

Skills are defined in `.claude/skills/` — read the relevant `SKILL.md` for the full step-by-step procedure.

---

## Plugin Identity

- **Display name:** Consentric — Simple Cookie Consent
- **WP.org slug:** `consentric-cookie-consent`
- **Slug (files):** `consentric-cookie-consent`
- **Main file:** `consentric-cookie-consent.php`
- **GitHub:** https://github.com/sergiorsmaster/consentric
- **Min WP:** 6.0 | **Min PHP:** 7.4
- **Text domain:** `consentric-cookie-consent`

---

## Code Conventions

- PHP prefix `cscc_` on all functions, classes, hooks, and options.
- JS namespace `window.SimpleCookieConsent`.
- Wrap all user-facing strings with `__()` / `esc_html__()` / `esc_html_e()`.
- WordPress Settings API for all admin forms.
- Enqueue scripts/styles via `wp_enqueue_scripts` / `admin_enqueue_scripts`.
- No inline styles in PHP templates — use CSS files or `wp_add_inline_style`.
- Do NOT use `load_plugin_textdomain()` — WordPress.org handles translations automatically since WP 4.6.

### WordPress Security Rules (Plugin Check compliance)

These rules are **mandatory** — Plugin Check will flag violations as errors or warnings.

**Output escaping** — every `echo` must use an escaping function:
- Text: `esc_html()`, `esc_html_e()`, `esc_html__()`
- Attributes: `esc_attr()`, `esc_attr_e()`
- URLs: `esc_url()`
- HTML fragments built in PHP: `wp_kses_post()`
- Numbers in inline JS: cast with `(int)` or `intval()`

**Input handling** — always `wp_unslash()` before sanitizing superglobals:
- `sanitize_text_field( wp_unslash( $_POST['field'] ) )`
- `sanitize_key( wp_unslash( $_GET['key'] ) )`
- `absint( $_GET['id'] )` (absint is safe without unslash)
- Nonces: `wp_verify_nonce( sanitize_text_field( wp_unslash( $_POST['nonce'] ) ), 'action' )`

**Nonce verification** — required before processing any `$_GET` / `$_POST` data:
- For form submissions: use `wp_nonce_field()` + `wp_verify_nonce()`
- For read-only display params (e.g. `$_GET['tab']`): add `// phpcs:ignore WordPress.Security.NonceVerification.Recommended` with a comment explaining why

**Redirects** — always use `wp_safe_redirect()` instead of `wp_redirect()`.

**Direct database queries** — our custom table (`cscc_cookies`) requires `$wpdb` calls. Add a phpcs ignore comment:
```php
// phpcs:ignore WordPress.DB.DirectDatabaseQuery, WordPress.DB.PreparedSQL.InterpolatedNotPrepared -- $table uses trusted prefix
$results = $wpdb->get_results( "SELECT * FROM {$table} WHERE ..." );
```

**No inline JS event handlers** — use `data-cscc-action` attributes + delegated JS listeners instead of `onclick`.

---

## Repository Structure

```
consentric/
├── consentric-cookie-consent.php ← Main file (plugin header + bootstrap + CSCC_VERSION constant)
├── uninstall.php               ← Drops table + deletes all cscc_* options
├── readme.txt                  ← WordPress.org listing
├── .github/workflows/
│   └── release.yml             ← Auto-build zip + GitHub Release on vX.X.X tag push
├── .claude/skills/             ← AI slash commands (skills)
├── includes/
│   ├── class-cscc-activator.php     ← create_tables(), set_defaults(), seed_own_cookie()
│   ├── class-cscc-deactivator.php
│   ├── class-cscc-consent-store.php ← PHP cookie reader
│   ├── class-cscc-cookie-scanner.php
│   ├── class-cscc-wp-consent-api.php
│   ├── class-cscc-polylang.php
│   └── class-cscc-shortcodes.php    ← [cscc_cookie_list], [cscc_preferences]
├── admin/
│   ├── class-cscc-admin.php         ← Menu, tabs, register_settings(), handle_cookie_actions()
│   ├── views/
│   │   ├── tab-general.php         ← cscc_general settings group
│   │   ├── tab-appearance.php      ← cscc_appearance settings group
│   │   ├── tab-jurisdiction.php    ← cscc_jurisdiction settings group
│   │   ├── tab-integrations.php    ← cscc_integrations settings group + Debug
│   │   ├── tab-cookies.php         ← Cookie list + scanner (custom form, no Settings API)
│   │   └── tab-help.php            ← Read-only reference (CSS, shortcodes, JS API)
│   └── assets/
│       ├── admin.css
│       └── admin.js
├── public/
│   ├── class-cscc-public.php        ← enqueue_scripts(), render_banner(), render_modal()
│   ├── views/
│   │   ├── banner.php
│   │   ├── modal.php
│   │   └── preferences-icon.php    ← Cookie SVG icon
│   └── assets/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sergiorsmaster/consentric](https://github.com/sergiorsmaster/consentric) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
