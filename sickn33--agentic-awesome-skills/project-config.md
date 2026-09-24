---
trigger: always_on
description: > This document is optimized for AI agents and LLMs. Rules are prioritized by performance impact.
---

# Postgres Best Practices

**Version 1.0.0**
Supabase
January 2026

> This document is optimized for AI agents and LLMs. Rules are prioritized by performance impact.

---

## Abstract

Comprehensive Postgres performance optimization guide for developers using Supabase and Postgres. Contains performance rules across 8 categories, prioritized by impact from critical (query performance, connection management) to incremental (advanced features). Each rule includes detailed explanations, incorrect vs. correct SQL examples, query plan analysis, and specific performance metrics to guide automated optimization and code generation.

---

## Table of Contents

1. [Query Performance](#query-performance) - **CRITICAL**
   - 1.1 [Add Indexes on WHERE and JOIN Columns](#11-add-indexes-on-where-and-join-columns)
   - 1.2 [Choose the Right Index Type for Your Data](#12-choose-the-right-index-type-for-your-data)
   - 1.3 [Create Composite Indexes for Multi-Column Queries](#13-create-composite-indexes-for-multi-column-queries)
   - 1.4 [Use Covering Indexes to Avoid Table Lookups](#14-use-covering-indexes-to-avoid-table-lookups)
   - 1.5 [Use Partial Indexes for Filtered Queries](#15-use-partial-indexes-for-filtered-queries)

2. [Connection Management](#connection-management) - **CRITICAL**
   - 2.1 [Configure Idle Connection Timeouts](#21-configure-idle-connection-timeouts)
   - 2.2 [Set Appropriate Connection Limits](#22-set-appropriate-connection-limits)
   - 2.3 [Use Connection Pooling for All Applications](#23-use-connection-pooling-for-all-applications)
   - 2.4 [Use Prepared Statements Correctly with Pooling](#24-use-prepared-statements-correctly-with-pooling)

3. [Security & RLS](#security-rls) - **CRITICAL**
   - 3.1 [Apply Principle of Least Privilege](#31-apply-principle-of-least-privilege)
   - 3.2 [Enable Row Level Security for Multi-Tenant Data](#32-enable-row-level-security-for-multi-tenant-data)
   - 3.3 [Optimize RLS Policies for Performance](#33-optimize-rls-policies-for-performance)

4. [Schema Design](#schema-design) - **HIGH**
   - 4.1 [Choose Appropriate Data Types](#41-choose-appropriate-data-types)
   - 4.2 [Index Foreign Key Columns](#42-index-foreign-key-columns)
   - 4.3 [Partition Large Tables for Better Performance](#43-partition-large-tables-for-better-performance)
   - 4.4 [Select Optimal Primary Key Strategy](#44-select-optimal-primary-key-strategy)
   - 4.5 [Use Lowercase Identifiers for Compatibility](#45-use-lowercase-identifiers-for-compatibility)

5. [Concurrency & Locking](#concurrency-locking) - **MEDIUM-HIGH**
   - 5.1 [Keep Transactions Short to Reduce Lock Contention](#51-keep-transactions-short-to-reduce-lock-contention)
   - 5.2 [Prevent Deadlocks with Consistent Lock Ordering](#52-prevent-deadlocks-with-consistent-lock-ordering)
   - 5.3 [Use Advisory Locks for Application-Level Locking](#53-use-advisory-locks-for-application-level-locking)
   - 5.4 [Use SKIP LOCKED for Non-Blocking Queue Processing](#54-use-skip-locked-for-non-blocking-queue-processing)

6. [Data Access Patterns](#data-access-patterns) - **MEDIUM**
   - 6.1 [Batch INSERT Statements for Bulk Data](#61-batch-insert-statements-for-bulk-data)
   - 6.2 [Eliminate N+1 Queries with Batch Loading](#62-eliminate-n1-queries-with-batch-loading)
   - 6.3 [Use Cursor-Based Pagination Instead of OFFSET](#63-use-cursor-based-pagination-instead-of-offset)
   - 6.4 [Use UPSERT for Insert-or-Update Operations](#64-use-upsert-for-insert-or-update-operations)

7. [Monitoring & Diagnostics](#monitoring-diagnostics) - **LOW-MEDIUM**
   - 7.1 [Enable pg_stat_statements for Query Analysis](#71-enable-pgstatstatements-for-query-analysis)
   - 7.2 [Maintain Table Statistics with VACUUM and ANALYZE](#72-maintain-table-statistics-with-vacuum-and-analyze)
   - 7.3 [Use EXPLAIN ANALYZE to Diagnose Slow Queries](#73-use-explain-analyze-to-diagnose-slow-queries)

8. [Advanced Features](#advanced-features) - **LOW**
   - 8.1 [Index JSONB Columns for Efficient Querying](#81-index-jsonb-columns-for-efficient-querying)
   - 8.2 [Use tsvector for Full-Text Search](#82-use-tsvector-for-full-text-search)

---

## 1. Query Performance

**Impact: CRITICAL**

Slow queries, missing indexes, inefficient query plans. The most common source of Postgres performance issues.

### 1.1 Add Indexes on WHERE and JOIN Columns

**Impact: CRITICAL (100-1000x faster queries on large tables)**

Queries filtering or joining on unindexed columns cause full table scans, which become exponentially slower as tables grow.

**Incorrect (sequential scan on large table):**

```sql
-- No index on customer_id causes full table scan
select * from orders where customer_id = 123;

-- EXPLAIN shows: Seq Scan on orders (cost=0.00..25000.00 rows=100 width=85)
```

**Correct (index scan):**

```sql
-- Create index on frequently filtered column
create index orders_customer_id_idx on orders (customer_id);

select * from orders where customer_id = 123;

-- EXPLAIN shows: Index Scan using orders_customer_id_idx (cost=0.42..8.44 rows=100 width=85)
-- Index the referencing column
create index orders_customer_id_idx on orders (customer_id);

select c.name, o.total
from customers c
join orders o on o.customer_id = c.id;
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sickn33/agentic-awesome-skills](https://github.com/sickn33/agentic-awesome-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
