---
trigger: always_on
description: QueryCanvas is a database client extension for VS Code/Cursor with AI-powered features.
---

# Cursor Rules for QueryCanvas - Database Client

## Project Overview
QueryCanvas is a database client extension for VS Code/Cursor with AI-powered features.

---

## ⚠️ MOST IMPORTANT: SQL Display Options Syntax

When generating SQL with display options, follow these rules EXACTLY:

### @column directive (cell styling)
```sql
@column <name> type=int if<0:color=red
```
- Use `if<operator><value>:style` for conditions
- Example: `@column 売上 type=int if<0:color=red if>1000000:color=green`

### @row directive (entire row styling)
```sql
@row <column_name><operator><value>:<styles>
```
- **NO `if` keyword!**
- Use `==` (double equals) for equality
- Quote strings: `"value"` or `'value'`
- Use `bg` NOT `background`
- Example: `@row 曜日=="土":bg=#eeeeff`
- Example: `@row 売上>1000000:bg=#ccffcc,bold=true`

### @chart directive (graph visualization) 🆕
```sql
@chart type=line x=日付 y=売上,利益
@chart type=mixed x=月 y=売上:bar,目標:line
@chart type=pie x=店舗名 y=件数 colors="#FF6384,#36A2EB,#FFCE56,#4BC0C0,#9966FF"
```
- **Required:** `type`, `x` (or `xAxis`), `y` (or `yAxis`)
- **Chart types:** `line`, `bar`, `pie`, `area`, `scatter`, `mixed`
- **Y-axis:** Comma-separated for multiple series (e.g., `y=店舗A,店舗B,店舗C`)
- **Mixed charts:** Specify type for each series: `y=売上:bar,目標:line,達成率:line`
- **Optional:** `title="タイトル"`, `legend=true`, `grid=true`, `stacked=true`, `curve=smooth`
- **Colors (for pie charts):** `colors="#color1,#color2,#color3"` - Specify colors for each segment
- Example: `@chart type=line x=日付 y=小村井店,京成小岩店 title="店舗別売上推移"`
- Example: `@chart type=mixed x=月 y=売上:bar,目標:line title="売上と目標"`
- Example: `@chart type=pie x=店舗名 y=件数 colors="#FF6384,#36A2EB,#FFCE56" title="店舗別シェア"`

### ❌ WRONG Examples
```sql
@row if 曜日=土:background=#eee     ❌ NO! Has 'if', uses '=', uses 'background'
@row 国名=フランス:bg=#fee           ❌ NO! No quotes, uses single '='
@chart x=日付                       ❌ NO! Missing required 'type' and 'y'
```

### ✅ CORRECT Examples
```sql
@row 曜日=="土":bg=#eee              ✅ YES!
@row 国名=="フランス":bg=#fee        ✅ YES!
@row 売上>1000000:bg=#ccffcc         ✅ YES!
@chart type=line x=日付 y=売上       ✅ YES!
```

---

## 🎨 SQL Display Options - Quick Reference

### Two Types of Styling

#### 1. Column Styling (`@column`) - Styles individual cells
```sql
/**
 * @column 売上 type=int align=right format=number comma=true if<0:color=red
 */
```
- Uses `if<value:style` syntax for conditional styling
- Applies to individual cells in that column

#### 2. Row Styling (`@row`) - Styles entire rows
```sql
/**
 * @row 曜日=="土":bg=#eeeeff
 * @row 売上>1000000:bg=#ccffcc,bold=true
 */
```
- **CRITICAL:** Do NOT use `if` keyword
- Use `==` (double equals) for equality, NOT `=` (single)
- Strings MUST be quoted: `"値"` or `'値'`
- Use `bg` or `backgroundColor`, NOT `background`
- Applies styling to the entire row

### Common Mistakes ⚠️

| ❌ WRONG | ✅ CORRECT |
|----------|-----------|
| `@row if 曜日=土:background=#eee` | `@row 曜日=="土":bg=#eee` |
| `@row 国名=フランス:bg=#fee` | `@row 国名=="フランス":bg=#fee` |
| `@row 売上>1000:background=#cfc` | `@row 売上>1000:bg=#cfc` |

### Syntax Comparison

| Feature | Column Style | Row Style |
|---------|-------------|-----------|
| Directive | `@column` | `@row` |
| Conditional | `if<0:color=red` | `売上<0:bg=#fcc` |
| `if` keyword | ✅ YES (use `if`) | ❌ NO (`if` not used) |
| Equality | `==` or single `=` | `==` (double only) |
| String values | No quotes needed in options | MUST quote: `"value"` |
| Example | `@column 損益 type=int if<0:color=red` | `@row 曜日=="土":bg=#eef` |

---
## 🤖 Cursor AI Integration

This extension is designed to work seamlessly with Cursor AI. You can edit SQL queries by modifying the session file.

### SQL Session File

**Location:** `.vscode/querycanvas-session.json`

**What it contains:**
- Current SQL query in the editor
- Active database connection ID
- Last update timestamp

**How Cursor AI can help:**

1. **Edit SQL directly in session file:**
   - User can ask: "Modify the SQL in the session to add a WHERE clause"
   - Cursor AI edits `.vscode/querycanvas-session.json`
   - Changes reflect in Database Client UI immediately (via file watcher)

2. **Generate optimized SQL:**
   - User can ask: "Rewrite this SQL with display options"
   - Cursor AI updates the `sqlInput` field with formatted SQL

3. **Add display options:**
   - User can ask: "Add display formatting to this query"
   - Cursor AI inserts `/** @column ... */` comments

**Example session file:**
```json
{
  "connectionId": "production-db",
  "isConnected": false,
  "sqlInput": "/**\n * @column amount align=right format=number comma=true\n */\nSELECT amount FROM orders",
  "lastUpdated": "2025-12-28T12:00:00.000Z"
}
```

**Workflow:**
```
User: "Add display options to format the 'price' column"
  ↓
Cursor AI: Edits .vscode/querycanvas-session.json
  ↓
File watcher: Detects change
  ↓
UI: Updates automatically
  ↓
User: Sees formatted SQL in Database Client
```

## SQL Display Options Feature

This extension supports custom display formatting through SQL comments.

### Syntax
Use `/** ... */` comments at the beginning or end of SQL queries with `@column` and `@row` directives:

```sql
/**
 * @column <column_name> <option>=<value> <option>=<value> ...

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [okuyamashin/querycanvas](https://github.com/okuyamashin/querycanvas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
