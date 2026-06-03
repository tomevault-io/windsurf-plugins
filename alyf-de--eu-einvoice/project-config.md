---
trigger: always_on
description: This is a Frappe Framework app for creating and importing European e-invoices with ERPNext.
---

# EU E-Invoice Copilot Instructions

This is a Frappe Framework app for creating and importing European e-invoices with ERPNext.

## Code Style & Formatting

### Python
- **Indentation**: Use tabs (width: 4 spaces) for Python files
- **Line length**: Max 110 characters (configured in ruff)
- **Target version**: Python 3.10+
- **Formatter**: Ruff (replaces Black)
- **Linter**: Ruff (replaces Flake8, isort, etc.)
- **Quote style**: Double quotes (`"`)
- **Line endings**: LF (Unix style)
- Always add final newline and trim trailing whitespace

### JavaScript/Vue
- **Indentation**: Use tabs (width: 4 spaces)
- **Line length**: Max 99 characters
- **Formatter**: Prettier for JS/Vue/SCSS files
- **Linter**: ESLint (extends `eslint:recommended`)
- **Line endings**: LF (Unix style)
- Always add final newline and trim trailing whitespace

### JSON
- **Indentation**: 2 spaces (not tabs)
- **No final newline** in JSON files

## Frappe Framework Rules

### Python API Best Practices
- ❌ `frappe.db.commit()` → ✅ Let framework handle transactions
- ❌ `print()` in DocTypes → ✅ `frappe.msgprint()` or `frappe.logger()`
- ❌ `map()`, `filter()` → ✅ List comprehensions
- ❌ `eval(expression)` → ✅ `frappe.safe_eval(expression)`
- ❌ `get_value("Single DocType", "Single DocType", field)` → ✅ `get_single_value("Single DocType", field)`
- ❌ Global DB calls → ✅ Wrap in functions (multitenancy)
- ❌ `frappe.cache().set/get()` → ✅ `frappe.cache().set_value/get_value()`

### DocType Controllers
- ❌ `def after_save():` → ✅ Use valid hooks: `after_insert`, `on_update`, `before_save`
- ❌ Modify without commit in post-save hooks: `self.status = "New"` → ✅ `self.db_set("status", "New")`
- ❌ Modify child tables while iterating → ✅ Iterate over copy of list

### Translations
- Always wrap user-facing strings in translation functions:
  - Python: `_("Text")` (underscore function)
  - JavaScript: `__("Text")` (double underscore function)
- ❌ `_("")` or `__("")` → ✅ Remove empty translations
- ❌ `_("{}")` or `_("{0}")` → ✅ Use variables directly
- ❌ `_("  text  ")` → ✅ `_("text")` (no trailing spaces)
- ❌ `_("Hello {}".format(name))` → ✅ `_("Hello {0}").format(name)`
- ❌ `_("Text") + _("More")` → ✅ `_("Text More")` (single translation)
- ❌ `_("Long " + "text")` → ✅ `_("Long text")` (no concatenation)

### JavaScript API Best Practices
- ❌ `cur_frm` → ✅ `frm` parameter
- ❌ `in_list(list, item)` → ✅ `list.includes(item)`
- ❌ `frappe.utils.debounce(fn, 300)()` → ✅ Create once: `const debounced = frappe.utils.debounce(fn, 300)`

### Query Builder
- ❌ `.orderby("field", "desc")` → ✅ `.orderby("field", order=frappe.qb.desc)`

## Frappe Bench Commands

### Bench Location

- **Local development**: Varies by installation
- **GitHub Actions CI**: `/home/runner/frappe-bench`
- **App location in CI**: `/home/runner/frappe-bench/apps/eu_einvoice`

### Development
```bash
# Start development server
bench start

# Open Python console with Frappe context
bench --site test_site console

# Execute Python code in Frappe context
bench --site test_site execute "frappe.clear_cache()"
bench --site test_site execute "module.function_name()"

# Clear cache
bench clear-cache
bench clear-website-cache
```

### Testing
```bash
# Run all tests for the app
bench --site test_site run-tests --app eu_einvoice

# Run tests for a specific module
bench --site test_site run-tests --module eu_einvoice.european_e_invoice.doctype.e_invoice_import.test_e_invoice_import

# Run specific test method
bench --site test_site run-tests --module eu_einvoice.european_e_invoice.doctype.e_invoice_import.test_e_invoice_import --test test_method_name
```

### Translation
```bash
# Generate POT file (translation template)
bench generate-pot-file --app eu_einvoice

# Update PO files from POT
bench update-po-files --app eu_einvoice

# Build translation files
bench compile-po-to-mo --app eu_einvoice
```

### Database
```bash
# Run migrations
bench --site test_site migrate

# Backup database
bench --site test_site backup

# Restore database
bench --site test_site restore /path/to/backup.sql.gz

# Database console
bench --site test_site mariadb
```

## Pre-commit Hooks

This repository uses pre-commit hooks that run automatically on `git commit`:
- **trailing-whitespace**: Removes trailing whitespace (except JSON, TXT, CSV, MD, SVG)
- **check-yaml**: Validates YAML syntax
- **check-merge-conflict**: Prevents committing merge conflict markers
- **check-ast**: Validates Python syntax
- **check-json**: Validates JSON syntax
- **check-toml**: Validates TOML syntax
- **debug-statements**: Prevents committing debug statements
- **ruff**: Lints and auto-fixes Python code
- **ruff-format**: Formats Python code
- **prettier**: Formats JS/Vue/SCSS files
- **eslint**: Lints JavaScript files
- **commitlint**: Validates commit messages (conventional commits)

### Commit Message Format
Follow [Conventional Commits](https://www.conventionalcommits.org/):
```
<type>(<scope>): <subject>

<body>

<footer>
```

Types: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `chore`, `ci`, `build`

Examples:
- `feat: add e-invoice import functionality`
- `fix(import): handle missing tax categories`
- `docs: update installation instructions`

## Project Structure

```
eu_einvoice/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alyf-de/eu_einvoice](https://github.com/alyf-de/eu_einvoice) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
