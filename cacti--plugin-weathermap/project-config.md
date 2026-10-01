---
trigger: always_on
description: Use these instructions for PHP changes in Cacti organization repositories. Repository-local `AGENTS.md`, `.github/copilot-instructions.md`, `CONTRIBUTING.md`, Composer scripts, and CI workflows take precedence when they are more specific.
---


# Cacti PHP instructions

Use these instructions for PHP changes in Cacti organization repositories. Repository-local `AGENTS.md`, `.github/copilot-instructions.md`, `CONTRIBUTING.md`, Composer scripts, and CI workflows take precedence when they are more specific.

## Establish the repository contract first

- Before editing, inspect the target branch, `composer.json`, CI matrix, test configuration, formatter configuration, and nearby code. Do not assume that Cacti core and every plugin support the same PHP or Cacti versions.
- Write syntax and dependencies for the lowest PHP version supported by the target repository and branch. For plugins, also respect the Cacti branch/version used by plugin CI.
- Preserve the repository's GPL header, copyright form, bootstrap pattern, changelog process, and established file organization. Do not perform unrelated cleanup or repository-wide formatting.
- Confirm that a reported defect exists on the target branch before fixing it. Cacti release branches and `develop` can differ substantially.

## PHP style

- Indent PHP with tabs, displayed at four spaces. Use Unix line endings, remove trailing whitespace, and leave one newline at end of file.
- Put opening braces on the same line for functions, classes, closures, and control structures.
- Prefer single-quoted strings. Use double quotes only when interpolation or escape handling makes them appropriate.
- Use short arrays (`[]`), lowercase PHP keywords and `true`, `false`, and `null`, and strict comparisons when the type is known.
- Put one space around concatenation and binary operators. Do not add spaces inside array offsets.
- Prefer `foreach` and current PHP constructs over removed legacy constructs. Prefer simple string functions when a regular expression is unnecessary; never introduce `ereg_*`.
- Preserve readable alignment already used in a touched block, but do not create large whitespace-only diffs.
- Use the repository's PHP CS Fixer configuration when present. Do not replace Cacti's established style with a generic PSR preset.

## Cacti architecture and entry points

- Web pages must bootstrap through the repository's established authenticated entry point, normally `include/auth.php`; do not bypass authentication or initialize Cacti piecemeal.
- CLI scripts must use the established CLI bootstrap, normally `include/cli_check.php`.
- Use Cacti headers, footers, form helpers, table helpers, URL helpers, and plugin APIs instead of duplicating framework behavior or emitting a parallel UI.
- Follow the common page flow where applicable: initialize defaults, validate request variables, dispatch the action, then render through Cacti's header/footer helpers.
- Plugin changes must preserve the plugin `INFO` metadata and registration conventions. Register hooks and realms through the plugin APIs. AJAX plugin URLs normally include `header=false` when the surrounding Cacti layout must be suppressed.
- Reuse shared functions and existing domain helpers before adding a new abstraction. Keep behavior-preserving refactoring limited to the area already being changed.

## Input, authorization, CSRF, and output

- Treat all request, session, database, device, SNMP, file, and command output as untrusted at the relevant boundary.
- Do not read `$_GET`, `$_POST`, or `$_REQUEST` directly in new code. Use Cacti request helpers such as `get_filter_request_var()` / `gfrv()`, `get_request_var()` / `grv()`, `get_nfilter_request_var()` / `gnrv()`, and the repository's form validation helpers.
- Understand request caching: once `gfrv()` has validated a variable, later `grv()` or `gnrv()` calls for that variable retrieve the cached value. Do not add misleading duplicate validation or casts, but ensure every externally controlled variable has an appropriate validation point.
- Validate by allowlist and expected type, range, or format. Never rely on escaping as input validation.
- Enforce authorization on the server for every privileged page and action. Hiding a control in the UI is not authorization. For plugins, keep realm registration, upgrade/repair behavior, and action-level permission checks consistent.
- Protect state-changing forms and AJAX requests with Cacti's CSRF mechanism. AJAX posts include `__csrf_magic: csrfMagicToken` unless a repository-local helper supplies it.
- Escape at output time for the exact context. Use Cacti's HTML text and attribute helpers; use `html_escape_attr()` for attribute contexts. Do not use a text-context escape helper for JavaScript, JSON, URL, or HTML attributes.
- Build local URLs with `cacti_url()` where available and redirect with `cacti_redirect()`. Do not redirect to unvalidated user-controlled destinations.
- Encode JSON with the repository's safe JSON conventions and return the correct content type. Do not build JSON by concatenating strings.
- Wrap user-visible strings in `__()` and pass the plugin text domain where the surrounding plugin does so.

## Database access


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Cacti/plugin_weathermap](https://github.com/Cacti/plugin_weathermap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
