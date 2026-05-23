---
trigger: always_on
description: This document provides a comprehensive set of guidelines for SQL development, covering aspects from naming conventions and table design to query optimization, security, and testing, aimed at promoting best practices in database management and application development.
---

# SQL Development Guidelines

This document provides a comprehensive set of guidelines for SQL development, covering aspects from naming conventions and table design to query optimization, security, and testing, aimed at promoting best practices in database management and application development.

## Implementing These Principles

These guidelines are built upon the following core principles:

- Principle 1: Clarity and Maintainability: Ensuring SQL code and database structures are easy to understand, modify, and maintain.
- Principle 2: Performance and Scalability: Designing databases and queries for optimal performance and the ability to handle growing data and user loads.
- Principle 3: Data Integrity and Security: Protecting data accuracy, consistency, and safeguarding against unauthorized access or loss.
- Principle 4: Consistency: Applying uniform standards across the database schema and SQL code.

## Table of contents

- Rule 1: Naming Conventions
- Rule 2: Table Design
- Rule 3: Query Writing
- Rule 4: Indexing Strategy
- Rule 5: Security Guidelines
- Rule 6: Performance Optimization Tips
- Rule 7: Transaction Guidelines
- Rule 8: Migration Best Practices
- Rule 9: Code Examples
- Rule 10: Testing Guidelines
- Rule 11: Monitoring Practices

## Rule 1: Naming Conventions

Title: Consistent Naming for Database Objects
Description: Defines standard naming conventions for database objects like tables, columns, foreign keys, views, triggers, indexes, and constraints to ensure clarity and consistency across the database schema.

**Good example:**

**General:**
- Use snake_case for all database objects
- Use plural for table names (e.g., `users`, `orders`)
- Use singular for column names (e.g., `user_id`, `order_date`)
- Prefix foreign keys with referenced table name (e.g., `user_id`)
- Use verb_noun format for stored procedures (e.g., `get_user`, `update_order`)

**Prefixes:**
- Views: `v_`
- Triggers: `trg_`
- Indexes: `idx_`
- Constraints:
  - Primary Key: `pk_`
  - Foreign Key: `fk_`
  - Unique: `uq_`
  - Check: `ck_`

## Rule 2: Table Design

Title: Best Practices for Designing Database Tables
Description: Outlines best practices for table design, including primary keys, appropriate data types, normalization (aiming for at least 3NF), inclusion of timestamp columns (`created_at`, `updated_at`), use of foreign key constraints for referential integrity, consideration for soft deletes (`deleted_at`), and strategic indexing for frequently queried columns.

**Good example:**

**Best Practices:**
- Always include a primary key.
- Use appropriate data types for columns.
- Normalize to at least 3NF unless there's a good reason not to.
- Include `created_at` and `updated_at` timestamp columns.
- Use foreign key constraints to maintain referential integrity.
- Consider soft deletes using a `deleted_at` timestamp.
- Add appropriate indexes for frequently queried columns.

**Common Columns:**
- `id: BIGINT AUTO_INCREMENT PRIMARY KEY`
- `created_at: TIMESTAMP DEFAULT CURRENT_TIMESTAMP`
- `updated_at: TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP`
- `deleted_at: TIMESTAMP NULL`

## Rule 3: Query Writing

Title: Guidelines for Writing Effective and Readable SQL Queries
Description: Provides guidelines for formatting SQL queries for readability and optimizing them for performance. This includes advice on `SELECT` statements, using `EXISTS` versus `IN`, appropriate indexing, avoiding correlated subqueries where possible, using `EXPLAIN` to analyze query performance, considering query execution plans, and using batch operations for bulk data modifications.

**Good example:**

**Formatting:**
- Use uppercase for SQL keywords (e.g., `SELECT`, `FROM`, `WHERE`).
- Place one clause per line for better readability.
- Indent subqueries and Common Table Expressions (CTEs).
- Align column lists vertically in `SELECT` statements.
- Use meaningful and concise table aliases.

**Performance:**
- Avoid `SELECT *`; specify only the columns needed.
- Use `EXISTS` instead of `IN` with subqueries for potentially better performance, especially on large datasets.
- Ensure appropriate indexes are available for columns used in `WHERE` clauses, `JOIN` conditions, and `ORDER BY` clauses.
- Avoid correlated subqueries when an equivalent join or non-correlated subquery can be used.
- Use `EXPLAIN` (or similar command for your RDBMS) to analyze query performance and understand the execution plan.
- Consider the query execution plan to identify bottlenecks.
- Use batch operations for bulk updates or inserts to reduce overhead.

**Example Query:**
```sql
SELECT
    u.id,
    u.first_name,
    u.last_name,
    o.order_date
FROM
    users u
LEFT JOIN
    orders o ON u.id = o.user_id
WHERE
    u.deleted_at IS NULL
    AND o.status = 'COMPLETED'
ORDER BY
    o.order_date DESC;
```

## Rule 4: Indexing Strategy

Title: Principles for Effective Database Indexing

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jabrena/cursor-rules-spring-boot](https://github.com/jabrena/cursor-rules-spring-boot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
