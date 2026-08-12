---
trigger: always_on
description: This repo is a monolithic PHP + MySQL web app (OOU Bursary Store inventory). The goal of these instructions is to get an AI coding agent immediately productive with minimal assumptions.
---

# Bursary Inventory — Copilot Instructions (concise)

This repo is a monolithic PHP + MySQL web app (OOU Bursary Store inventory). The goal of these instructions is to get an AI coding agent immediately productive with minimal assumptions.

- Project entry: `index.php` is the login + role dispatch entrypoint. Role strings include `admin`, `vc`, `storekeeper`.
- DB connection: `assets/inc/config.php` exposes a global `$mysqli`. Use it — do NOT create ad-hoc new DB connectors.
- Sessions: code relies on `$_SESSION['user_id']`, `$_SESSION['username']`, `$_SESSION['role']`, `$_SESSION['full_name']`. Protect pages with `require_once 'assets/inc/checklogin.php'`.

- Conventions to follow:
  - Use prepared statements via `$mysqli->prepare()`, bind params, call `$stmt->execute()` and `$stmt->close()`.
  - Call `log_action($mysqli, $user_id, $message, $meta)` from `assets/inc/functions.php` for auditable changes.
  - Layout: include `assets/inc/head.php` first, then `nav.php`/`sidebar*.php`, then page body, then `assets/inc/footer.php`.
  - UI: use SweetAlert via `assets/js/swal.js` for notifications, and existing JS initializers under `assets/js/pages/` for component behavior.

- Typical request flow (pattern to mirror): page (GET) → form POST to same or helper endpoint → server validates → prepared statement → `log_action()` → redirect/render.

- Common files to check first:
  - `index.php` — login + redirects
  - `assets/inc/config.php` — DB and environment toggles
  - `assets/inc/functions.php` — helpers including `log_action()`
  - `assets/inc/checklogin.php` — session enforcement
  - `assets/inc/head.php` — common head, active-sidebar JS, SWAL hooks
  - `sql/migrations/` — DB schema/migrations

- Local dev / run:
  - XAMPP-style: start Apache + MySQL, open `http://localhost/bursary/index.php`.
  - Default DB name used in config: `bursary`. Local dev commonly uses `root` without a password.

- Debugging & safety:
  - Temporarily enable `display_errors` and `error_reporting(E_ALL)` in `assets/inc/config.php` only for local debugging; do not commit credential changes.
  - Sanitize and validate uploaded CSVs in `upload_stock.php` before DB writes.

- Tasks/PR checklist for agents:
  1. Reuse the global `$mysqli` and prepared-statement pattern.
  2. Preserve session variable names and `role` values.
  3. Update or add pages following the include/layout pattern (`head.php` → `nav/sidebar` → body → `footer.php`).
  4. Add `log_action()` calls for state-changing operations.
  5. Avoid editing `assets/inc/config.php` unless changing only non-secret flags; never commit production credentials.

- Helpful snippets (copy/paste):

Prepared statement
```php
$stmt = $mysqli->prepare("INSERT INTO items (name, qty) VALUES (?, ?)");
$stmt->bind_param('si', $name, $qty);
$stmt->execute();
$stmt->close();
```

Protect page
```php
require_once 'assets/inc/checklogin.php';
// now $_SESSION enforced
```

If you need additional details (example queries, specific page scaffolds, or test data), tell me which area to expand and I will update this file with a short example block.
```instructions
<!-- Project-specific Copilot instructions for AI coding agents -->
# Bursary Inventory — Copilot Instructions

This repository is a PHP + MySQL web app (OOU Bursary Store inventory). Use these concise rules to make changes safely and productively.

- Project entry & auth:
  - The login entry is [index.php](index.php). Authentication uses `password_hash()`/`password_verify()` and role strings (e.g. `admin`, `vc`, `storekeeper`). See the login flow in [index.php](index.php).

- Database & DB access:
  - DB connection lives in [assets/inc/config.php](assets/inc/config.php). The app uses a global `$mysqli` instance.
  - Common tables: `users` (fields: `user_id`, `username`, `password`, `role`, `full_name`) and `logs` (used by `log_action()`). See [assets/inc/functions.php](assets/inc/functions.php).
  - Prefer prepared statements (`$mysqli->prepare()`) — the project already uses them in `index.php`. When adding queries, follow the same style and call `->close()` on statements.

- Includes & layout conventions:
  - Shared partials live under `assets/inc/` (e.g. [head.php](assets/inc/head.php), `nav.php`, `sidebar.php`, `footer.php`). New pages follow the pattern: include `head.php`, then `nav.php`/`sidebar.php`, then page content, then `footer.php`.
  - Static assets served from `assets/` (CSS, JS, images). Use relative paths like `assets/js/...` as existing pages do.

- Session & access control:
  - Sessions are used heavily — check `$_SESSION['user_id']` / `$_SESSION['role']`. Some pages use `assets/inc/checklogin.php` to enforce login. When adding protected pages, include the existing check login pattern.

- UI & alerts:
  - UI messages use SweetAlert (`assets/js/swal.js`) and small JS injections in `head.php` / `index.php`. Mirror that approach for consistent UX.

- Testing / running locally:
  - This is an XAMPP-style app: start Apache + MySQL and open `http://localhost/bursary/index.php`.
  - DB name used by default: `bursary` (see `assets/inc/config.php`). Expect `root` user w/o password in local dev.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [De-Icon1/php_bursarystore-](https://github.com/De-Icon1/php_bursarystore-) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
