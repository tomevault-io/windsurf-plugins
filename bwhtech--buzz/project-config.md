---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

At the start of every session, read `.claude/memory/memory.md` to load project context.
After completing significant work (new patterns, architectural decisions, solved problems),
update `.claude/memory/memory.md`. Keep it under 300 lines — summarize when it grows.

---

## This defines the best practices to write backend code in the Frappe Framework

* Frappe Framework is a full-stack web application framework that contains all the necessary components for building modern web applications.
* It provides background workers using Redis, real-time updates using sockets, and a database layer using MariaDB.
* Bench is the official command-line tool for managing Frappe applications.

## Backend Development

### JSON & Request Handling

* Always use built-in functions for parsing JSON:

  * `frappe.parse_json` (handles dicts, lists, and JSON strings safely)

* Never use `json.loads` directly on request data.

* For outbound HTTP requests (calling external APIs), use:

  * `frappe.integration.utils.make_get_request`
  * `frappe.integration.utils.make_post_request`
  * `frappe.integration.utils.make_put_request`
  * `frappe.integration.utils.make_patch_request`

### Datatype Conversion & Utilities

* For converting datatypes (e.g. str → int, str → float, etc.) use built-in helpers:

  * `frappe.utils.data.cint`
  * `frappe.utils.data.cstr`
  * `frappe.utils.data.flt`
  * `frappe.utils.data.getdate`
  * `frappe.utils.data.get_datetime`

* `frappe.utils.data` contains most conversion and formatting helpers you will ever need:

  * date / datetime parsing
  * currency formatting
  * number formatting

* Do NOT create custom utility functions for these conversions.

* If unsure, ask before implementing.

### DocType Access Patterns

* When fetching an existing DocType, prefer:

  * `frappe.get_cached_doc`

* Use `frappe.get_doc` when:

  * creating a new document

  * To create a new doc go to bench console via bench --site sitename console and use frappe.new_doc("DocType") and then create the doc, don't create the doc via json as the validations doesn't run


### Optimization

* Don't use get_doc or get_cached_doc inside for loop it creates n+1 db problem use frappe.get_all with all the params required and then loop over that list

### Database Access

* Prefer ORM methods:

  * `frappe.get_all`
  * `frappe.get_list`
  * `frappe.db.get_value`

* Avoid raw SQL absolutely.


### Permissions & Security

* Always respect user permissions.
* Use `ignore_permissions=True` only when absolutely required and justified.

### Background Jobs & Performance

* For long-running or heavy operations, always use:

  * `frappe.enqueue`

* Never block request-response cycles with heavy business logic.

### Error Handling & Logging

* Use `frappe.throw` or specific exceptions like `frappe.ValidationError` for user-facing errors.
* Use `frappe.log_error` for unexpected or system-level exceptions.
* Avoid bare `except:` blocks.

### General Guidelines

* Prefer framework conventions over custom implementations.
* Keep business logic out of controllers where possible.
* Write readable, predictable, and maintainable code.



## Frontend Development

1. Always use async/await; avoid callback-based patterns and nested promises.

2. Use Frappe-provided APIs for server calls: `frappe.call` with `async: true`. Prefer Promise-based usage over callbacks.

3. Use Frappe's global JS helpers instead of native JS equivalents:
   * `cstr()` instead of `String()`
   * `cint()` instead of `parseInt()`
   * `flt()` instead of `parseFloat()`
   * `is_null()` instead of manual null/undefined/empty checks
   * `format_currency()` for currency formatting


## Crawling

Always use gemini as much as possible for getting the context, to get the help use gemini --help

For checking if the site works you can use the agent-browser use agent-browser --help to get the context for it


## Commands

### Frontend (Dashboard)
```bash
# dev server
yarn dev  # or: cd dashboard && yarn dev

# build for production
yarn build  # outputs to buzz/public/dashboard + buzz/www/dashboard.html

# lint/format frontend
cd dashboard && yarn lint
```

### Backend (Python)

Always run bench migrate after doctype schema changes.

```bash
# linting/formatting (via pre-commit)
pre-commit run --all-files

# run ruff directly
ruff check buzz/
ruff format buzz/

# install app to site
bench --site [site-name] install-app buzz
```

Use bench --help to see how to work with frappe bench, e.g. bench execute, bench console, etc. are very useful

### Testing

There are unit tests, run using bench run-tests. Site name is buzz.localhost, but if not found, ask user for it. The credentials are Administrator/admin.

* To test in UI, use agent-browser.
* For frontend changes use :8080 since yarn dev server is running.
* Use in headed mode unless specified

## Architecture

**Three-tier stack:**
1. **Backend**: Frappe Framework (Python) - DocTypes, API, permissions, scheduler
2. **Dashboard**: Vue 3 + FrappeUI + Vite - attendee/sponsor/checkin UI

**Core entity**: `Buzz Event` DocType drives everything (tickets, sponsors, schedule, payments).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bwhtech/buzz](https://github.com/bwhtech/buzz) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-01 -->
