---
trigger: always_on
description: Type-safe SQL query builder with powerful SQL printers. Zero dependencies, tree-shakeable. Pure TypeScript, works everywhere.
---

# sumak

Type-safe SQL query builder with powerful SQL printers. Zero dependencies, tree-shakeable. Pure TypeScript, works everywhere.

> [!IMPORTANT]
> Keep `AGENTS.md` updated with project status.

## Architecture

7-layer pipeline: **Schema → Builder → AST → Plugin/Hook → Normalize (NbE) → Optimize (rewrite rules) → Printer → SQL**

```
User Code
  │
  ├─ sumak({ dialect, tables })     ← DB type auto-inferred
  │
  ├─ db.selectFrom("users")       ← TypedSelectBuilder<DB, "users", O>
  │    .select("id", "name")       ← O narrows to Pick<O, "id"|"name">
  │    .where(typedEq(...))        ← Expression<boolean> enforced
  │    .build()                    ← SelectNode (frozen AST)
  │
  ├─ db.compile(node)             ← Full pipeline:
  │    1. Plugin AST transforms
  │    2. Lifecycle hooks (before)
  │    3. Normalize (NbE)           ← Predicate simplification, constant folding
  │    4. Optimize (rewrite rules)  ← Predicate pushdown, subquery flattening
  │    5. Printer → SQL
  │    6. Plugin query transforms
  │    7. Lifecycle hooks (after)
  │
  └─ { sql, params }              ← Parameterized output
```

## Project Structure

```
src/
  sumak.ts                   # sumak() factory + Sumak<DB> class
  index.ts                  # Main API — all exports
  pg.ts                     # Sub-path: sumak/pg
  mssql.ts                  # Sub-path: sumak/mssql
  mysql.ts                  # Sub-path: sumak/mysql
  sqlite.ts                 # Sub-path: sumak/sqlite
  schema.ts                 # Sub-path: sumak/schema
  errors.ts                 # Custom error classes
  types.ts                  # Shared types (CompiledQuery, SQLDialect, etc.)
  schema/
    types.ts                # ColumnType<S,I,U>, Selectable, Insertable, Updateable
    column.ts               # ColumnBuilder<S,I,U>, 22 column factories (serial, text, etc.)
    table.ts                # defineTable(), InferTable
    type-utils.ts           # Nullable, ResolveColumnType, SelectResult, etc.
    index.ts                # Re-exports
  ast/
    nodes.ts                # ~40 AST node types (discriminated unions, frozen)
    expression.ts           # Untyped expression factories (col, lit, eq, etc.)
    typed-expression.ts     # Expression<T> phantom types (typedEq, typedCol, etc.)
    visitor.ts              # ASTVisitor interface + visitNode dispatcher
    transformer.ts          # ASTTransformer base class
  builder/
    select.ts               # SelectBuilder (untyped, immutable)
    insert.ts               # InsertBuilder
    update.ts               # UpdateBuilder
    delete.ts               # DeleteBuilder
    merge.ts                # MergeBuilder
    expression.ts           # Expression builder helpers (val, resetParamCounter)
    raw.ts                  # Raw SQL escape hatch
    typed-select.ts         # TypedSelectBuilder<DB, TB, O>
    typed-insert.ts         # TypedInsertBuilder<DB, TB>
    typed-update.ts         # TypedUpdateBuilder<DB, TB>
    typed-delete.ts         # TypedDeleteBuilder<DB, TB>
    typed-merge.ts          # TypedMergeBuilder<DB, Target, Source>
  printer/
    base.ts                 # BasePrinter — visitor-based SQL generation
    pg.ts                   # PgPrinter ($1 params, double-quote identifiers)
    mssql.ts                # MssqlPrinter (@p0 params, square-bracket identifiers)
    mysql.ts                # MysqlPrinter (? params, backtick identifiers)
    sqlite.ts               # SqlitePrinter (? params, double-quote identifiers)
    formatter.ts            # SQL pretty-printer (keyword-aware)
    document.ts             # Wadler-style document algebra (text/line/nest/group/render)
    types.ts                # Printer, PrinterOptions, PrintMode
  dialect/
    pg.ts                   # pgDialect() factory
    mssql.ts                # mssqlDialect() factory
    mysql.ts                # mysqlDialect() factory
    sqlite.ts               # sqliteDialect() factory
    types.ts                # Dialect interface
  plugin/
    types.ts                # SumakPlugin interface
    plugin-manager.ts       # PluginManager — sequential plugin pipeline
    hooks.ts                # Hookable — lifecycle hooks (query:before/after, etc.)
    with-schema.ts          # WithSchemaPlugin — auto schema prefix
    soft-delete.ts          # SoftDeletePlugin — auto WHERE deleted_at IS NULL
    camel-case.ts           # CamelCasePlugin — snake_case → camelCase results
  normalize/
    types.ts                # CNF type, NormalizeOptions
    expression.ts           # NbE: normalizeExpression, toCNF, fromCNF
    query.ts                # normalizeQuery — applies NbE to all query types
    index.ts                # Re-exports
  optimize/
    types.ts                # RewriteRule interface, OptimizeOptions
    rules.ts                # Built-in rules: predicate pushdown, subquery flattening
    optimizer.ts            # optimize() — normalize + rewrite rules to fixpoint
    index.ts                # Re-exports
  utils/
    identifier.ts           # Identifier quoting per dialect
    param.ts                # Parameter formatting per dialect
test/
  sumak.test.ts              # Integration: sumak() clean API, plugins, hooks

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [productdevbook/sumak](https://github.com/productdevbook/sumak) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
