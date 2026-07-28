---
trigger: always_on
description: This file provides guidance for AI agents working with Dolt databases to maximize productivity and follow best practices.
---

# AGENT.md - Dolt Database Operations Guide

This file provides guidance for AI agents working with Dolt databases to maximize productivity and follow best practices.

## Quick Start

Dolt is "Git for Data" - a SQL database with version control capabilities. All Git commands have Dolt equivalents:
- `git add` → `dolt add`  
- `git commit` → `dolt commit`
- `git branch` → `dolt branch`
- `git merge` → `dolt merge`
- `git diff` → `dolt diff`

For help and documentation on commands, you can run `dolt --help` and `dolt <command> --help`.

## Essential Dolt CLI Commands

### Repository Operations
```bash
# Initialize new database
dolt init

# Clone existing database
dolt clone <remote-url>

# Show current status
dolt status

# View commit history
dolt log
```

### Branch Management
```bash
# List branches
dolt branch

# Create new branch
dolt branch <branch-name>

# Switch branches
dolt checkout <branch-name>

# Create and switch to new branch
dolt checkout -b <branch-name>
```

### Checkout Behavior with Running SQL Servers
- `dolt checkout` on the CLI only affects the shell process that runs the command. When a `dolt sql-server` is running, existing SQL connections keep their current branch until they explicitly switch.
- Each SQL session (CLI `dolt sql`, MySQL client, application connection) maintains its own active branch. Run `CALL dolt_checkout('<branch>');` at the beginning of every session or scripted block to ensure you are on the correct branch.
- Chain branch changes inside scripts: start with `CALL dolt_checkout('<branch>');`, then run your queries. Do not assume a previous checkout persists for new connections.
- When automating, include the checkout in the same transaction / session context where the data changes execute.
- A good way to make sure a `dolt sql` session connects to the br1 branch for instance is `dolt --branch br1 sql`.

### Data Operations
```bash
# Stage changes
dolt add <table-name>
dolt add .  # stage all changes

# Commit changes
dolt commit -m "commit message"

# View differences
dolt diff
dolt diff <table-name>
dolt diff <branch1> <branch2>

# Merge branches
dolt merge <branch-name>
```

## Starting and Connecting to Dolt SQL Server

### Start SQL Server
```bash
# Start server on default port (3306)
dolt sql-server

# Start on specific port
dolt sql-server --port=3307

# Start with specific host
dolt sql-server --host=0.0.0.0 --port=3307

# Start in background
dolt sql-server --port=3307 &
```

### Connecting to SQL Server
```bash
# Connect with dolt sql command
dolt sql

# Connect with mysql client
mysql -h 127.0.0.1 -P 3306 -u root

# Connect with specific database
mysql -h 127.0.0.1 -P 3306 -u root -D <database-name>
```

## Dolt Testing with dolt_test System Table

### Unit Testing with dolt_test

The dolt_test system table provides a powerful way to create and run unit tests for your database. This is the preferred method for testing data integrity, business rules, and schema validation.

#### Creating Tests

Tests are created by inserting rows into the `dolt_tests` system table:

```sql
-- Create a simple test
INSERT INTO `dolt_tests` VALUES (
    'test_user_count', 
    'validation', 
    'SELECT COUNT(*) as user_count FROM users;', 
    'row_count',
    '>',
    '0'
);

-- Create a test with expected result
INSERT INTO `dolt_tests` VALUES (
    'test_valid_emails', 
    'validation', 
    'SELECT COUNT(*) FROM users WHERE email NOT LIKE "%@%";', 
    'row_count',
    '==',
    '0'
);

-- Create a schema validation test
INSERT INTO `dolt_tests` VALUES (
    'test_users_schema', 
    'schema', 
    'DESCRIBE users;', 
    'row_count',
    '>=',
    '5'
);
```

#### Test Structure

Each test row contains:
- test_name: Unique identifier for the test
- test_group: Optional grouping for tests (e.g., 'validation', 'schema', 'integration')
- test_query: SQL query to execute
- assertion_type: Type of assertion ('expected_rows', 'expected_columns', 'expected_single_value')
- assertion_comparator: Comparison operator ('==', '>', '<', '>=', '<=', '!=')
- assertion_value: Expected value for comparison

#### Running Tests

```sql
-- Run all tests
SELECT * FROM dolt_test_run();

-- Run specific test
SELECT * FROM dolt_test_run('test_user_count');

-- Run tests with filtering
SELECT * FROM dolt_test_run() WHERE test_name LIKE 'test_user%' AND status != 'PASS';
```

#### Test Result Interpretation

The dolt_test_run() function returns:
- test_name: Name of the test
- status: PASS, FAIL, or ERROR
- actual_result: Actual query result
- expected_result: Expected result
- message: Additional details

#### Advanced Testing Examples

```sql
-- Test data integrity
INSERT INTO `dolt_tests` VALUES (
    'test_no_orphaned_orders', 
    'integrity', 
    'SELECT COUNT(*) FROM orders o LEFT JOIN users u ON o.user_id = u.id WHERE u.id IS NULL;', 
    'row_count',
    '==',
    '0'
);

-- Test business rules
INSERT INTO `dolt_tests` VALUES (
    'test_positive_prices', 
    'business_rules', 
    'SELECT COUNT(*) FROM products WHERE price <= 0;', 
    'row_count',
    '==',
    '0'
);

-- Test complex relationships
INSERT INTO `dolt_tests` VALUES (
    'test_order_totals', 
    'integrity', 

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dolthub/dolt](https://github.com/dolthub/dolt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
