---
trigger: always_on
description: This file provides guidance to ai agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to ai agents when working with code in this repository.

## Repository Overview

This is VillageSQL Server, a fork of MySQL 8.4.6 LTS that adds VillageSQL Extension Framework (VEF), which includes support for custom types. The codebase generally follows Oracle/MySQL development practices, though with additional VillageSQL modifications.

**Terminology:**
- **VEF** (VillageSQL Extension Framework): The overall framework for building and loading extensions
- **VEB** (VillageSQL Extension Bundle): The file format for VillageSQL extension packages (`.veb` files)

**Development Guides:**
- [Adding SQL Commands](Docs/ADDING_SQL_COMMANDS.md) - Complete checklist for adding new SQL commands to the parser
- [Adding System Tables](villagesql/schema/ADDING_SYSTEM_TABLES.md) - Guide for adding new VillageSQL system tables to victionary
- [Error Handling](Docs/ERROR_HANDLING.md) - strategies for handling errors in VillageSQL and the boundary with MySQL
- [CI Build Cache](.github/CI_BUILD_CACHE.md) - How CI build caching works, known issues, and diagnostics

## Key Development Commands

### Environment Setup

Create an `AGENTS.local.md` file (ignored by git) with your local environment variables for AI agents to reference. You can also create `~/AGENTS.local.md` for settings that apply across all your projects.

**IMPORTANT**: Before running any build or test commands, always read `AGENTS.local.md` in the repository root to get the correct environment variable values for this machine. Both files are mentioned at the bottom to ensure they are read.

Example AGENTS.local.md:
```
HOME=/your/home/path
BUILD_HOME=$HOME/build
SOURCE_HOME=$HOME/code/villagesql-server
```

### Building
Assume environment variable definitions as per AGENTS.local.md

**Determining CPU cores for parallel builds:**
- `getconf _NPROCESSORS_ONLN` (works on both macOS and Linux)
- Typical machines have 8-16 cores, use with `-j` flag (e.g., `-j14` for 14 cores), but check what number to use. All examples here use -j14.

```bash
# Standard release build
cd $BUILD_HOME
rm CMakeCache.txt; cmake $SOURCE_HOME
make -j14

# Debug build
rm CMakeCache.txt; cmake $SOURCE_HOME -DWITH_DEBUG=1 -DWITH_SHOW_PARSE_TREE=1
make -j14

# Maintainer build (strict warnings)
rm CMakeCache.txt; cmake $SOURCE_HOME -DMYSQL_MAINTAINER_MODE=ON
make -j14

# macOS with specific SSL
rm CMakeCache.txt; cmake $SOURCE_HOME -DWITH_SSL=/opt/homebrew/opt/openssl@3
make -j14
```

### Testing
```bash
# Run all tests
cd $BUILD_HOME
./mysql-test/mysql-test-run.pl

# Run specific test
./mysql-test/mysql-test-run.pl test_name

# Run tests in parallel (faster)
./mysql-test/mysql-test-run.pl --parallel=auto

# Run ALL VillageSQL tests including sub-suites (alter_table, create_table, etc.)
./mysql-test/mysql-test-run.pl --do-suite=village --nounit-tests --parallel=auto

# Run VillageSQL tests including "big" tests (longer running)
./mysql-test/mysql-test-run.pl --do-suite=village --nounit-tests  --parallel=auto --big-test

# Run specific VillageSQL sub-suite
./mysql-test/mysql-test-run.pl --suite=villagesql/alter_table --parallel=auto

# Run specific test suite
./mysql-test/mysql-test-run.pl --suite=innodb --parallel=auto

# Run all available test suites
./mysql-test/mysql-test-run.pl --suite=all --parallel=auto

# Run multiple specific suites
./mysql-test/mysql-test-run.pl --suite=villagesql,innodb --parallel=auto

# Run tests with increased server logging verbosity (needed to see LogVSQL output)
./mysql-test/mysql-test-run.pl --suite=villagesql --parallel=auto --mysqld=--log-error-verbosity=3

# Create/update test results
./mysql-test/mysql-test-run.pl --record my_new_test

# Build and run villagesql unittests
make -j14 villagesql-unit-tests && ctest -L villagesql
```

#### Debugging Test Failures and Error Messages

When debugging failed tests or investigating server-side logging:

**Error Log Locations:**
- Main error log: `$BUILD_HOME/mysql-test/var/log/mysqld.1.err`
- Test-specific logs: `$BUILD_HOME/mysql-test/var/log/test_name/`

**Finding VillageSQL Debug Messages:**
VillageSQL uses `LogVSQL()` for debug logging, which appears in the MySQL error log but not in test output by default. To view these messages:

```bash
# View error log during/after test run
tail -f $BUILD_HOME/mysql-test/var/log/mysqld.1.err

# Search for VillageSQL debug messages
grep -i "villagesql" $BUILD_HOME/mysql-test/var/log/mysqld.1.err
```

**Including Error Log Messages in Test Output:**
If you need error log messages to appear in test output for debugging, add a perl section to read from the error log:

```perl
--perl
use strict;
my $error_log = $ENV{'MYSQLTEST_VARDIR'} . "/log/mysqld.1.err";
open(FILE, "$error_log") or die("Unable to open error log: $error_log");
while (<FILE>) {
  my $line = $_;
  if ($line =~ /VillageSQL.*DEBUG/) {
    # Remove timestamp/thread info, show just the message
    $line =~ s/^\d{4}-\d{2}-\d{2}T\d{2}:\d{2}:\d{2}\.\d+Z\s+\d+\s+\[ERROR\]\s+\[MY-\d+\]\s+\[Server\]\s+//;
    print $line;
  }
}
close(FILE);
EOF
```

If you need to add your own temporary debugging, use sql_print_information to make it easier to know it is
temporary and needs to be removed.

#### Test Result Portability


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [villagesql/villagesql-server](https://github.com/villagesql/villagesql-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
