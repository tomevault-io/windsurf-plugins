---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Fatal Plugin Auto Deactivator** (slug: `fatal-plugin-auto-deactivator`) is a WordPress.org-distributed plugin that automatically deactivates any plugin causing a fatal PHP error, logs the incident, and shows a custom error page instead of the white screen of death. Plain PHP — no build step, no test suite, no node/npm. No REST endpoints, no AJAX. Since 1.5.0: two cron events (watchdog + alert drain) and one public filter (`fpad_watchdog_interval`).

Detailed docs live in `docs/` (excluded from distribution; keep them updated when behavior changes):
- `docs/feature-map.md` — **feature → file/function lookup, sync pairs, playbooks for common changes; start here when locating or changing a feature**
- `docs/architecture.md` — entry points, drop-in mechanism, error flow, execution contexts
- `docs/reference.md` — option schemas, hook table, nonces/admin surfaces, per-class method reference
- `docs/development.md` — setup, conventions, manual test scenarios
- `docs/deployment.md` — release checklist and CI

- Minimum: WordPress 5.3, PHP 7.0 (the drop-in mechanism requires WP ≥ 5.2; code must stay PHP 7.0 compatible).
- Code follows WordPress Coding Standards (tabs, Yoda conditions, `esc_html`/`esc_url` escaping, nonces). `phpcs:ignore` annotations are used deliberately — keep them.

## Architecture: the drop-in mechanism

The whole plugin revolves around WordPress's **drop-in** system. On activation, `includes/fatal-error-handler-dropin.php` is copied to `wp-content/fatal-error-handler.php`. WordPress core loads that drop-in during its shutdown handler when a fatal error occurs and calls `handle()` on the object it returns — this works even when the crashing plugin took the whole request down.

Flow on a fatal error:
1. WP core shutdown handler loads `wp-content/fatal-error-handler.php` (the drop-in).
2. Drop-in defines `FPAD_PLUGIN_DIR` relative to its own location, requires `includes/class-fatal-error-handler.php` from this plugin, defines `QM_DISABLE_ERROR_HANDLER` (Query Monitor conflict), and returns `new FPAD_Fatal_Error_Handler()`.
3. `FPAD_Fatal_Error_Handler::handle()` reads `error_get_last()`, matches the error file path against each active plugin's directory (prefix match on the error file path — not stack trace analysis), calls `deactivate_plugins()` if matched, **always** records the incident in `fpad_deactivation_log` (even when no plugin matched), renders an inline HTML 500 page, and exits.

**Critical constraint**: `FPAD_Fatal_Error_Handler` and the drop-in run in a context where WordPress may be only partially loaded. Every WP function call in that class must be guarded with `function_exists()` / file includes, as the existing code does. Never add unguarded WP API calls, hooks, or plugin-loaded assumptions to `class-fatal-error-handler.php` or the drop-in. **This includes constants**: core registers the fatal handler at `wp-settings.php`'s `wp_register_fatal_error_handler()`, *before* it loads `formatting.php` (escaping), `functions.php` → `option.php` (options API), and `wp_plugin_directory_constants()` (`WP_PLUGIN_DIR`) — so a fatal in an early drop-in (`advanced-cache.php`, `object-cache.php`, `db.php`, `sunrise.php`) hits this class without them, and on PHP 8 an undefined constant throws. Guard constants with `defined()`. **For functions, `function_exists()` is necessary but NOT sufficient**: when the fatal is inside `wp_start_object_cache()` (a broken `object-cache.php` — the most common case), core never reaches `wp-includes/cache.php`, so `wp_cache_get()` is undefined and everything built on the options API *exists but throws* — `get_option()` directly, and `esc_html()`/`sanitize_text_field()`/`get_bloginfo()` indirectly via `wp_check_invalid_utf8()`'s `get_option( 'blog_charset' )`. Route such calls through `FPAD_Fatal_Error_Handler::wp_call( $fn, $args, $fallback )`, escape page output with the pure-PHP `esc()`/`esc_link()`, and keep each step of `handle()` in its own `try/catch`.

### Drop-in lifecycle (three reinstall paths — keep them in sync)

The drop-in file must always exist and reference a valid class file inside this plugin's directory:
- `FPAD_Plugin_Lifecycle::activate()/deactivate()/uninstall()` — install/remove via `FPAD_Dropin_Manager`.
- `FPAD_Plugin_Lifecycle::check_dropin()` on `admin_init` — reinstalls if missing; a *foreign* drop-in is reclaimed at most once per 24 h via `may_reclaim_dropin()`, which shares the watchdog's back-off state so admin page loads cannot fight another plugin for the slot.
- `FPAD_Utils::plugin_upgrade_hook()` on `upgrader_process_complete` — removes and reinstalls the drop-in when *this* plugin is updated (the plugin dir is wiped during update, which would strand the drop-in's `require`).

`FPAD_Dropin_Manager::remove_dropin()`/`is_dropin_installed()` identify "our" drop-in by searching its content for the string `FPAD_Fatal_Error_Handler` — never remove a foreign drop-in.

### Classes (all in `includes/`, no autoloader — required explicitly in the main file)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rudlinkon/fatal-plugin-auto-deactivator](https://github.com/rudlinkon/fatal-plugin-auto-deactivator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
