---
trigger: always_on
description: Pluckr is a declarative read-query layer for ActiveRecord: a `Pluckr::Query`
---

# AGENTS.md

Pluckr is a declarative read-query layer for ActiveRecord: a `Pluckr::Query`
subclass declares the shape of the data, and Pluckr compiles it into exactly one
SQL statement returning frozen, ActiveRecord-free result objects.

Ruby 3.2+, ActiveRecord 7.1+ (6.1 also passes the suite), PostgreSQL / MySQL /
SQLite.

## Layout

```
lib/pluckr/
  query.rb                   class-level API: source, schema, find, for, fetch, first, to_sql
  relation.rb                immutable chaining + WhereChain, delegates root filtering to AR
  batch.rb                   Pluckr.batch - ad-hoc aggregates over caller-supplied relations
  schema/definition.rb       evaluates the DSL, validates, builds the AST
  schema/scope.rb            one level of the AST (the nodes selected for one model)
  schema/{field,one,exists,aggregate,node}.rb   AST nodes
  schema/conditions.rb       shared `where:` handling (Hash or callable)
  reflection/association.rb  the only place that touches AR reflection
  compiler/sql.rb            AST + Arel -> one SQL statement
  result/builder.rb          flat row -> nested frozen objects, with type casting
  result/object.rb           generated readers, to_h/to_hash, as_json, inspect, ==
  aliases.rb                 SQL alias encoding, shared by compiler and builder
  errors.rb                  Pluckr::Error and its descendants
```

Pipeline: `DSL -> schema AST -> reflection -> SQL compiler -> flat row -> result object`.

## DSL surface

`source`, `schema`, and inside a schema: `field` (`as:`), `one` (`via:`, nests),
`first`/`last` (`via:`, `order:`, fields only),
`exists` and `count`/`sum`/`avg`/`min`/`max` (`as:`, `via:`, `from:`, `column:`,
`where:`, `scope:`; `average` aliases `avg`). Query classes answer `where`/`where.not`/`order`/`limit`/
`offset`/`find`/`find_by`/`for`/`first`/`last`/`take`/`fetch`/`count`/`exists?`/
`one?`/`many?`/`find_each`/`in_batches`/`explain`/`to_sql`/`source_model`
(`first`/`last`/`take` also take a count; all three have a bang version).
`first`/`last`/`take`/`one?`/`many?` are defined rather than left to Enumerable,
which would have read the whole chain to answer them. On a chain that already
pages they follow AR exactly: `first(n)` clamps to the page (`find_nth_with_limit`),
`last` reads the page and takes its tail (`find_last`), `one?`/`many?` go through
`limited_count` (`limit_value ? count : limit(2).count`) - and `take(n)` replaces
the limit, because AR's does. `find` takes ids or an array of them, sends a block
to Enumerable, and treats `find(nil)` as `RecordNotFound`, all like AR; the
several-ids form goes through `for_ids`, so it needs `field` on the primary key
the way `.for` and batching do, while a single id does not.
`count`/`size`/`exists?`/`any?`/`none?`/`empty?` are delegated to the root AR
relation and compile no nodes - Enumerable would have answered them by building
every result object first. A block or a pattern argument goes back to Enumerable
(`return super if args.any? || block_given?`), mirroring
`ActiveRecord::Relation#any?`. `find_each`/`in_batches` seek by primary key
(`WHERE id > <last>`, never `OFFSET`), reorder by it, need `field` on it, and
refuse a chain that already has a `limit`/`offset`.
`.for` takes one AR record, a collection of them, or a relation of the source
model: one SQL statement, collections aligned to input order (requires `field`
on the primary key). An unloaded relation becomes a subquery, so its rows are
never instantiated: only its conditions survive (`UNUSED_RELATION_VALUES` drops
`select`/`order`/preloads/`lock`). A `limit`/`offset` cannot survive that (the
subquery loses the ORDER BY the page depends on, and MySQL rejects
`IN (SELECT ... LIMIT ...)`), so a paginated relation is plucked to primary keys
first and read by id: two statements, nothing instantiated. That pluck keeps the
joins the order may depend on (`PAGINATED_RELATION_VALUES` drops only
`select`/`lock`); a relation that joins to preload (`eager_loading?` or
`includes_values`) is loaded instead, because its `LIMIT` counts parent rows and
`pluck` would cut the page mid fan-out - see `page_keys`. Keys read
non-strictly - a relation is a set of conditions, so
rows the query filters out are absent rather than `RecordNotFound`, which stays
the answer only for records the caller handed over. Only `group`/`having` raises.
Not an association on the AR model - never `has_pluckr`.

Every read of a compiled read model goes through `Pluckr.select_all`
(`lib/pluckr.rb`), which wraps `connection.select_all` in a `fetch.pluckr`
notification carrying `:name`, `:sql` and `:rows`. Add execution paths there, not
around it. Statements Pluckr does not compile stay ActiveRecord's and publish
nothing: `count`/`exists?` and the `pluck` a paginated `.for` runs first.

`Pluckr.batch` is the second entry point: it takes relations the caller already
built and wraps each one in a derived table, so ActiveRecord's own scoping
applies and none of the reflection rules below are involved. It reuses the
Aggregate node (with `relation:` instead of `model:`), the compiler's
`aggregate_only_sql` and the result builder - no separate execution path.

## How the SQL is built

- plain fields -> selected columns, never `SELECT users.*`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [igorkasyanchuk/pluckr](https://github.com/igorkasyanchuk/pluckr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-15 -->
