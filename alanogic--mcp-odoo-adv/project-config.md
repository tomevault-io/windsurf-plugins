---
trigger: always_on
description: This file is the quick reference for AI assistants (Claude, Cursor, ChatGPT-via-MCP, others) talking to the Odoo MCP server. Humans should read [README.md](./README.md) instead.
---

# AGENTS.md — using this MCP server as an AI assistant

This file is the quick reference for AI assistants (Claude, Cursor, ChatGPT-via-MCP, others) talking to the Odoo MCP server. Humans should read [README.md](./README.md) instead.

## What this server gives you

| Surface | Count | Purpose |
|---|---|---|
| **Tools** | 3 | `execute_method`, `batch_execute`, `add_cookbook_pattern` |
| **Resources** | 11 | discovery (`odoo://models`, `odoo://model/{m}/schema`, …) + `odoo://cookbook/patterns` |
| **Prompts** | 3 | `search-customers`, `create-sales-order`, `odoo-exploration` |

You don't need a tool per Odoo operation. `execute_method` lets you call **any** method on **any** model — that's the entire Odoo ORM.

## The two universal tools

### `execute_method(model, method, args_json, kwargs_json)`

Call any Odoo method. Both `args_json` and `kwargs_json` are **JSON strings**, not Python dicts.

```python
execute_method(
    model="res.partner",
    method="search_read",
    args_json='[[["customer_rank", ">", 0]]]',
    kwargs_json='{"fields": ["name", "email"], "limit": 50}'
)
# → {"success": True, "result": [...]}
```

Every response is `{"success": bool, "result": ...}` or `{"success": False, "error": "..."}`. Always check `success` before touching `result`.

### `batch_execute(operations, atomic=True)`

Run multiple operations in one transaction. Use `@N` to reference a previous operation's result (1-indexed).

```python
batch_execute(
    operations=[
        {"model": "res.partner", "method": "create",
         "args_json": '[{"name": "Acme"}]'},
        {"model": "sale.order", "method": "create",
         "args_json": '[{"partner_id": "@1", "order_line": [[0, 0, {"product_id": 5, "product_uom_qty": 1}]]}]'},
        {"model": "sale.order", "method": "action_confirm",
         "args_json": '[[@2]]'}
    ],
    atomic=True
)
```

If any step fails with `atomic=True`, all previous steps roll back.

## Discovery resources

Read these **before guessing**. They cost less than a failed `execute_method` call.

| Resource | Returns |
|---|---|
| `odoo://models` | All Odoo models in this instance |
| `odoo://model/{model}` | Model summary + field list |
| `odoo://model/{model}/schema` | Full schema: fields, types, requireds, relationships |
| `odoo://model/{model}/access` | Your CRUD permissions on this model |
| `odoo://fields/{model}` | Field definitions only |
| `odoo://methods/{model}` | Methods available on this model |
| `odoo://workflows` | Business workflows enabled by installed modules |
| `odoo://server/info` | Odoo version, database, installed modules |
| `odoo://record/{model}/{id}` | A single record |
| `odoo://search/{model}/{domain}` | Search results for a quoted domain |

## The cookbook (after your first failure)

When `execute_method` returns an unexpected result or error, **read this resource before retrying**:

```
odoo://cookbook/patterns
```

It returns the **Learned Patterns** section of `COOKBOOK.md` — recipes that took ≥4 failed attempts in past sessions to discover. Examples: many2many search needs `in` with a list (not `=`), `mail.message.message_type` distinguishes drafts from real emails.

## Writing back to the cookbook

If you tried **≥4 distinct approaches** and finally found a working solution worth remembering, document it:

```python
add_cookbook_pattern(
    problem="Brief description of what you were trying to do",
    failed_approaches=[
        "First attempt — why it failed",
        "Second attempt — why it failed",
        "Third attempt — why it failed",
        "Fourth attempt — why it failed"
    ],
    working_solution='execute_method(...)',
    why_it_works="The technical reason this is correct",
    key_lesson="The portable takeaway",
    related_links=""  # optional
)
```

The tool **refuses** patterns with fewer than 4 failed approaches — this guards against documenting trivial misses. After a successful add, announce: `✅ New pattern documented: <key lesson>`.

## Decision flow

```
User asks for Odoo data/action
  │
  ├── Need to know what fields exist? → odoo://model/{m}/schema
  ├── Need to know if you have permission? → odoo://model/{m}/access
  └── Otherwise → execute_method
        │
        ├── success → return result
        │
        └── unexpected result → read odoo://cookbook/patterns
              │
              ├── matching recipe → apply it
              │
              └── no recipe → try next approach
                    │
                    └── after ≥4 failures → call add_cookbook_pattern
```

## Hard-won rules (if you read nothing else)

1. **Many2many fields require `in` with a list**, never `=` with a scalar. Wrong: `[["category_id", "=", 5]]`. Right: `[["category_id", "in", [5]]]`.

2. **Always pass `fields` to `search_read`.** Without it, every column of every record is returned — including huge text blobs and computed fields. Costs explode silently.

3. **Smart limit is 100/1000.** Without an explicit `limit`, you get 100 records and a warning. Hard cap is 1000 — to go higher, paginate.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AlanOgic/mcp-odoo-adv](https://github.com/AlanOgic/mcp-odoo-adv) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-18 -->
