---
trigger: always_on
description: MiniSQL is an embedded, single-file SQL database written in Go, inspired by SQLite. It implements a hand-written recursive-descent + state-machine SQL parser, a B+ tree storage engine with 4 KB pages, an LRU page cache, a Write-Ahead Log (WAL) for crash recovery, single-writer enforcement (via an atomic counter) for write transactions, and in-memory MVCC snapshot isolation for read-only transactions. It registers itself as a `database/sql` driver.
---

# AGENTS.md — MiniSQL Codebase Guide for AI Coding Agents

## Project Overview

MiniSQL is an embedded, single-file SQL database written in Go, inspired by SQLite. It implements a hand-written recursive-descent + state-machine SQL parser, a B+ tree storage engine with 4 KB pages, an LRU page cache, a Write-Ahead Log (WAL) for crash recovery, single-writer enforcement (via an atomic counter) for write transactions, and in-memory MVCC snapshot isolation for read-only transactions. It registers itself as a `database/sql` driver.

**Module:** `github.com/RichardKnop/minisql`
**Go version:** 1.26
**Not production-ready.** Treat it as a research/learning project.

**Feature summary:** INSERT/SELECT/UPDATE/DELETE, ON CONFLICT DO NOTHING/DO UPDATE, UNION/UNION ALL, DISTINCT, GROUP BY/HAVING, ORDER BY (multi-column), LIMIT/OFFSET, INNER/LEFT/RIGHT JOIN (arbitrary chain topology), CASE WHEN, CAST, INTERVAL, arithmetic, scalar functions, subqueries (non-correlated scalar + IN/NOT IN in WHERE, derived tables in FROM, CTEs, correlated UPDATE FROM), CHECK/FOREIGN KEY/NOT NULL/UNIQUE constraints, RETURNING, EXPLAIN/EXPLAIN ANALYZE, VACUUM, ANALYZE, PRAGMA, prepared statements (`?` placeholders), data types BOOLEAN/INT4/INT8/REAL/DOUBLE/TEXT/VARCHAR/TIMESTAMP/JSON/UUID, B-tree indexes (primary, unique, secondary, composite, covering, partial, expression), full-text inverted index, JSON inverted index, parallel full table scans, slow query logging.

---

## Repository Layout

```
/
├── minisql.go              # Driver, Conn, Stmt — database/sql/driver registration
├── tx.go                   # Tx — database/sql/driver.Tx
├── rows.go                 # Rows, Result — database/sql/driver.Rows / .Result
├── connection_string.go    # Connection string parameter parsing
├── go.mod / go.sum
│
├── internal/
│   ├── minisql/            # Core database engine (~140 .go files, listed selectively below)
│   │   │
│   │   │  ── Statement & Planning ──
│   │   ├── stmt.go               # Statement struct, all statement kinds, Clone(), Validate(), Prepare()
│   │   ├── stmt_join.go          # JoinClause, join statement helpers
│   │   ├── stmt_result.go        # StatementResult + lazy Iterator pattern
│   │   ├── condition.go          # WHERE condition evaluation (checkCondition, likeMatch, etc.)
│   │   ├── condition_node.go     # ConditionNode tree + ToDNF()
│   │   ├── expr.go               # Expr struct + Eval: arithmetic, CASE WHEN, functions
│   │   ├── compare.go            # compareValues helper used by sort and condition evaluation
│   │   ├── composite_key.go      # CompositeKey for multi-column indexes
│   │   │
│   │   │  ── Query Execution ──
│   │   ├── database.go           # Top-level Database: parse → validate → execute dispatch
│   │   ├── database_schema.go    # Schema introspection helpers (createTableDDL, createIndexDDL)
│   │   ├── database_options.go   # DatabaseOption functional options
│   │   ├── table.go              # Table struct: columns, indexes, query plan entry-point
│   │   ├── table_options.go      # TableOption functional options
│   │   ├── table_pager.go        # Table-level pager wiring
│   │   ├── table_primary_key.go  # Primary key B-tree operations
│   │   ├── table_secondary_index.go  # Secondary/unique index DML helpers
│   │   ├── table_unique_index.go # Unique index enforcement
│   │   ├── insert.go             # INSERT execution (incl. ON CONFLICT)
│   │   ├── select.go             # SELECT execution: selectStreaming + selectWithSort paths
│   │   ├── update.go             # UPDATE execution
│   │   ├── update_from.go        # UPDATE FROM (correlated subquery via context injection)
│   │   ├── delete.go             # DELETE execution
│   │   ├── returning.go          # RETURNING clause projection for INSERT/UPDATE/DELETE
│   │   ├── subquery.go           # Non-correlated subquery pre-evaluation (resolveSubqueries)
│   │   ├── correlated_subquery.go # Correlated subquery execution for UPDATE FROM
│   │   ├── derived_table.go      # FROM subquery: materialises into VirtualTable
│   │   ├── cte.go                # WITH clauses: CTE registry via context, VirtualTable injection
│   │   ├── check.go              # CHECK constraint evaluation
│   │   ├── foreign_key.go        # FOREIGN KEY enforcement, FK callbacks, CASCADE/SET NULL
│   │   ├── explain.go            # EXPLAIN / EXPLAIN ANALYZE output
│   │   ├── query_plan.go         # Query planner: scan-type selection, index optimisation
│   │   ├── query_plan_order.go   # ORDER BY planning (index skip-sort, heap, full sort)
│   │   ├── query_plan_join.go    # JOIN planning: flattenJoinTree, hash join selection
│   │   ├── query_plan_stats.go   # ANALYZE statistics: equi-depth histograms, selectivity
│   │   ├── hash_join.go          # Hash join build/probe executor
│   │   ├── parallel_scan.go      # Parallel full table scan (PRAGMA parallel_scan)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RichardKnop/minisql](https://github.com/RichardKnop/minisql) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-02 -->
