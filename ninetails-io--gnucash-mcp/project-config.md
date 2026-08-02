---
trigger: always_on
description: AI-facing orientation for contributors (human or AI) working on this
---

# GnuCash MCP Server — Contributor Guide

AI-facing orientation for contributors (human or AI) working on this
codebase. See `README.md` for end-user usage and `CHANGELOG.md` for
the per-release history.

Local working notes specific to the maintainer live in `CLAUDE.local.md`
(not in version control).

---

## Project at a glance

An MCP server that exposes a GnuCash SQLite book to AI assistants as a
set of typed tools. Read and write transactions, run reports, manage
scheduled transactions, budgets, investment lots, and a full business
module (customers, vendors, employees, invoices, bills).

**Tech stack:**
- Python 3.10+
- [piecash](https://github.com/sdementen/piecash) — GnuCash SQLite ORM
- [MCP Python SDK](https://github.com/modelcontextprotocol/python-sdk) (`mcp[cli]`)
- SQLAlchemy under piecash; direct Core access where the ORM blocks us
- pytest for unit + integration coverage

---

## How the codebase reached its current shape

A few architectural moves shaped the present structure. The full
release-by-release history is in `CHANGELOG.md`; what follows is the
arc, not the chronicle.

- **Single-file → modular mixins.** Early versions kept everything in
  one `book.py`. As the surface grew (reconciliation, reporting,
  budgets, scheduling, investments, business), the monolith was split
  into per-area mixins composed via `build_book_class`. Tools moved
  alongside under `tools/<area>.py`. Disabling a module via
  `--modules` skips both the mixin and the tool registration cleanly.
- **Lazy load with a registry.** `TOOL_MODULES` in `server.py` is the
  single source of truth for which tools belong to which module.
  `_lazy_load_tool_module(name)` imports `tools/<name>.py` only when
  the module is enabled, then `_apply_module_filter` removes anything
  registered that isn't in `TOOL_MODULES[<name>]`. A test in
  `tests/test_modules.py::TestToolFileVsModulesMapping` locks the
  contract — it catches the bug class where a new tool gets the
  `@mcp.tool()` decoration but is forgotten in `TOOL_MODULES`.
- **Multi-currency rebuild.** Originally USD-was-everywhere assumed.
  When the first non-USD-default book showed up, a class of bugs
  surfaced where reports summed raw `split.quantity` across
  commodities, prices defaulted to USD even for currency arguments,
  and FX gain/loss from rate drift wasn't recognized. The fix
  threaded `_split_in_default_currency` (or equivalent factor map)
  through every aggregation path, made `create_price` default to the
  book's default currency, and added realized FX recognition on
  cross-currency invoice payment.
- **Dashboard as a work queue.** `get_book_summary` started as a
  status snapshot and evolved into the LLM's first-call orientation
  surface. It now surfaces net-worth trajectory, runway, monthly net
  income, budget pacing, reconciliation backlog with split counts,
  warnings, and upcoming scheduled transactions — answering "what do
  I need to do next" in one call.
- **Audit log dispatcher.** Originally a 380-line if/elif chain;
  flattened into a `(entity_type, operation) → formatter` dispatch
  table. Adding a new entity-operation pair is one row in the table
  plus a small formatter function.

---

## Architecture

### Layered design

```
server.py              FastMCP bootstrap. TOOL_MODULES registry.
                       Lazy-loads tool modules from an enabled set;
                       disabled modules never build their Pydantic
                       schemas. Multi-book: GNUCASH_BOOK_PATH takes
                       an os.pathsep-separated list; a module-level
                       singleton holds the CURRENT book and the
                       inline switch_book tool repoints it (visible
                       only when 2+ books are configured).

tools/<area>.py        MCP tool registration. Thin wrappers that
                       validate schemas, unpack arguments (date
                       strings → date objects, etc.), call book
                       methods, format results.

book/<area>.py         Business-logic mixins composed into
                       GnuCashBook via build_book_class. One mixin
                       per subject area (core, business, budgets,
                       investments, reconciliation, reporting,
                       scheduling, admin, backup).

book/_base.py          BaseGnuCashBook. Shared helpers: open(),
                       _find_account, _resolve_account, _resolve_guid,
                       _unique_prefix, audit staging, write
                       verification, template-account filtering.

logging_config.py      Audit log + debug log. @audit_log decorator
                       wraps tool registrations; a dispatch table
                       keyed on (entity_type, operation) handles
                       formatting.

_format.py             Layer-neutral helpers (_format_number,
                       _apply_limit) used by both book and tools.
```

The mixin composition is deliberate. Each mixin owns its subject area
and can be enabled or disabled at server start via `--modules`.
`build_book_class` composes only the enabled mixins into a concrete
`GnuCashBook`, and `tools/` registration mirrors that — a disabled
module contributes zero tools to the MCP surface.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ninetails-io/gnucash-mcp](https://github.com/ninetails-io/gnucash-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
