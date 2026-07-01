---
trigger: always_on
description: This file is the primary onboarding document for resuming work on this project. Read it fully before making any changes.
---

# Joomla 3.x (JoomlaWorks Security Distribution) — Agent Onboarding & Patch Log

---

## For the AI Agent — Read This First

This file is the primary onboarding document for resuming work on this project. Read it fully before making any changes.

### What this project is

A security-hardened, PHP 8.x-compatible fork of Joomla 3.10.20 eLTS, maintained by JoomlaWorks (Fotis Evangelou). It is **not** an official Joomla release. The goal is to keep Joomla 3.x sites running safely on modern PHP and MySQL/MariaDB versions, with backported CVE fixes from Joomla 4/5/6.

- **GitHub repo:** https://github.com/joomlaworks/joomla-3.x
- **Current version:** Joomla 3.13.0 (released May 31, 2026)
- **Minimum PHP:** 7.4 — **all code changes must remain compatible with PHP 7.4**
- **Tested up to PHP:** 8.5
- **Database support:** MySQL 5.7+, MySQL 8.x, MariaDB, PostgreSQL, SQL Azure

### Key conventions to follow

- **PHP 7.4 minimum:** The `?Type` nullable syntax is fine (valid since PHP 7.1). The `#[\AllowDynamicProperties]` attribute is fine (parsed as a comment on PHP 7.x). Never use PHP 8.0+ syntax that would be a parse error on 7.4.
- **No comments unless the WHY is non-obvious.** Do not narrate what code does.
- **Security patches:** Always check `libraries/vendor/joomla/filter/src/InputFilter.php` (vendor-level) AND `libraries/src/Filter/InputFilter.php` (CMS-level) — they are separate implementations and both may need patching.
- **SQL changes** affect three files: `installation/sql/mysql/joomla.sql`, `installation/sql/postgresql/joomla.sql`, `installation/sql/sqlazure/joomla.sql`. Schema migration SQL goes in `administrator/components/com_admin/sql/updates/{mysql,postgresql,sqlazure}/`. File naming: `{version}-{YYYY-MM-DD}.sql` (e.g. `3.12.0-2026-05-21.sql`).
- **Changelog:** `CHANGELOG.md` is the detailed log; `README.md` has a brief per-version summary. Always update both.
- **AGENTS.md** (this file): append a new section for every version worked on, documenting every change made. This is how future sessions resume without losing context.

### What already exists and must NOT be re-applied

The following fixes are already in the codebase. Do not duplicate them:

- CVE-2025-54476 + H-1 extension (whitespace stripping + `\r\v\f` in `checkAttribute()`)
- CVE-2025-63083 (pagebreak toc.php htmlspecialchars)
- CVE-2026-21629 (com_ajax guest check)
- H-2 through H-8 (joomlaupdate ACL, TOTP timing, Yubikey HMAC, restore.php, RSS escaping, password reset timing, eval() removal)
- U-1 through U-3 (MediaHelper blocklist, sniff offset, images/.htaccess)
- P-1 through P-16 (all PHP 8.x compat fixes documented in the 3.11 section below)
- CVE-2025-63082 (data: URI blocking in checkAttribute)
- CVE-2024-40747 (ModuleHelper chrome attribs escaping)
- CVE-2025-25226 (quoteNameStr null byte/backslash rejection)
- CVE-2026-21631 (com_associations edit.php data-title escaping)
- All `#[\AllowDynamicProperties]` additions (Table, CMSObject, idna_convert)
- All remaining implicit-nullable fixes in fof/, vendor/joomla/session, vendor/joomla/data, vendor/joomla/di, vendor/google/recaptcha, vendor/symfony/yaml, vendor/joomla/filesystem, plugins/privacy/*
- All `(boolean)` → `(bool)` casts (libraries/src/ and libraries/joomla/, 13 files)
- `Uri::getInstance()` null guard (null → 'SERVER' coercion)
- `Uri::getInstance()` `HTTP_HOST` absent-in-CLI guard (`$httpHost` variable with `'localhost'` fallback before the SERVER URI branch — both Apache and IIS paths)
- All Q-3 through Q-10 PHP 8.x fixes (see 3.13 patch log): null guards in `HtmlView::escape()`, `Date::__construct()`, `ListModel::populateState()` (×2), `utf8_ltrim/rtrim/trim`, `Json::stringToObject()`; declared properties `$registeredurlparams` on `CMSApplication`, `$itemTags` on `TagsHelper`, `$empty` and `$dates` on `FinderIndexerQuery`
- `fixSchemas()` in `com_admin/script.php` (auto-runs SQL migrations + syncs `#__schemas` + syncs `manifest_cache.version` on upgrade)
- `administrator/manifests/files/joomla.xml` version and `<updateservers>` URL (updated; do not revert)
- `deleteUnexistingFiles()` 3.12 entries (beez3, hathor, eos310, phpversioncheck directories and language files)
- `com_joomlaupdate` OPcache fix (`cleanup()` reads version from `joomla.xml`, `complete.php` uses session value)

### Update server

The update server uses a **two-file architecture** that mirrors `update.joomla.org` exactly. The update site type in `#__update_sites` is `collection`, which means Joomla uses `CollectionAdapter` to parse `list.xml`. `CollectionAdapter` only understands `<extensionset>/<extension>` tags — it will silently return nothing if given an `<updates>` file.

- **`docs/list.xml`** — `<extensionset>` collection. Each `<extension>` entry has a `targetplatformversion` regex matched against `JVERSION` on the visitor's site, and a `detailsurl` pointing to `extension.xml`. Add one entry per supported minor version.
- **`docs/extension.xml`** — `<updates>` details file. Contains the `<update>` block with the download URL. `com_joomlaupdate` fetches this via `detailsurl` to display the "Update Now" button and download the package.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [joomlaworks/joomla-3.x](https://github.com/joomlaworks/joomla-3.x) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
