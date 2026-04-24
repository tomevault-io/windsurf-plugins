---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

OpenSearch SQL plugin — enables SQL and PPL (Piped Processing Language) queries against OpenSearch. This is a multi-module Gradle project (Java 21) that functions as an OpenSearch plugin.

## Build Commands

```bash
./gradlew build                          # Full build (compiles, tests, checks)
./gradlew build -x integTest            # Fast build (skip integration tests)
./gradlew :core:build                    # Build specific module
./gradlew test                           # Unit tests only
./gradlew :core:test --tests "*.AnalyzerTest"  # Single test class
./gradlew :integ-test:integTest          # Integration tests
./gradlew :integ-test:integTest -Dtests.class="*QueryIT"  # Single IT
./gradlew spotlessCheck                  # Check formatting
./gradlew spotlessApply                  # Auto-fix formatting
./gradlew generateGrammarSource          # Regenerate ANTLR parsers
```

## Code Style

- **Google Java Format** enforced via Spotless (2-space indent, 100 char line limit)
- **Lombok** is used throughout — `@Getter`, `@Builder`, `@RequiredArgsConstructor`, etc.
- **License header** required on all Java files (Apache 2.0). Missing headers fail the build.
- Pre-commit hooks run `spotlessApply` automatically
- All commits must include a DCO sign-off: `Signed-off-by: Name <email>` (use `git commit -s`).

## Architecture

### Query Pipeline

```
User Query (SQL/PPL)
  → Parsing (ANTLR) — produces parse tree
  → AST Construction (AstBuilder visitor) — produces UnresolvedPlan
  → Semantic Analysis (Analyzer) — resolves symbols/types → LogicalPlan
  → Planning (Planner + LogicalPlanOptimizer) — produces PhysicalPlan
  → Execution (ExecutionEngine) — streams ExprValue results
  → Response Formatting (ResponseFormatter — JSON/CSV/JDBC)
```

### Module Dependency Graph

```
plugin (OpenSearch plugin entry point, Guice DI wiring)
  ├── sql          — SQL parsing (ANTLR → AST via SQLSyntaxParser/AstBuilder)
  ├── ppl          — PPL parsing (ANTLR → AST via PPLSyntaxParser/AstBuilder)
  ├── core         — Central module: Analyzer, Planner, ExecutionEngine interfaces,
  │                  AST/LogicalPlan/PhysicalPlan node types, expression system, type system
  ├── opensearch   — OpenSearch storage engine, execution engine, client
  ├── protocol     — Response formatters (JSON, CSV, JDBC, YAML)
  ├── common       — Shared settings and utilities
  ├── legacy       — V1 SQL engine (backward compatibility fallback)
  ├── datasources  — Multi-datasource support (Glue, Security Lake, Prometheus)
  ├── async-query / async-query-core — Spark-based async query execution
  ├── direct-query / direct-query-core — Direct external datasource queries
  └── language-grammar — Centralized ANTLR .g4 grammar files
```

`core` has no dependency on other modules. `sql` and `ppl` depend on `core` and `language-grammar`. `opensearch` implements `core` interfaces.

### Key Source Locations

| Area | Key Files |
|------|-----------|
| Plugin entry | `plugin/.../SQLPlugin.java`, `plugin/.../OpenSearchPluginModule.java` |
| SQL parsing | `sql/.../sql/parser/AstBuilder.java`, `sql/.../SQLService.java` |
| PPL parsing | `ppl/.../ppl/parser/AstBuilder.java`, `ppl/.../PPLService.java` |
| ANTLR grammars | `language-grammar/src/main/antlr4/` (OpenSearchSQLParser.g4, OpenSearchPPLParser.g4) |
| Analysis | `core/.../analysis/Analyzer.java`, `core/.../analysis/ExpressionAnalyzer.java` |
| Planning | `core/.../planner/Planner.java`, `core/.../planner/logical/LogicalPlan.java` |
| Execution | `core/.../executor/ExecutionEngine.java`, `opensearch/.../OpenSearchExecutionEngine.java` |
| Storage | `opensearch/.../storage/OpenSearchStorageEngine.java` |
| Query orchestration | `core/.../executor/QueryService.java`, `core/.../executor/QueryPlanFactory.java` |

### Core Abstractions

- **`Node<T>`** — Base AST node with visitor pattern support
- **`UnresolvedPlan`** / **`LogicalPlan`** / **`PhysicalPlan`** — Query plan hierarchy (unresolved → logical → physical)
- **`Expression`** — Resolved expression with `valueOf()` and `type()`
- **`ExprValue`** — Runtime value types (ExprIntegerValue, ExprStringValue, etc.)
- **`ExprType`** — Type system (DATE, TIMESTAMP, DOUBLE, STRUCT, etc.)
- **`StorageEngine`** / **`Table`** — Pluggable storage abstraction
- **`ExecutionEngine`** — Executes physical plans, returns QueryResponse

### Design Patterns

- **Visitor pattern** used pervasively: `AbstractNodeVisitor`, `LogicalPlanNodeVisitor`, `PhysicalPlanNodeVisitor`, `ExpressionNodeVisitor`
- **PhysicalPlan** implements `Iterator<ExprValue>` for streaming execution
- **Guice** dependency injection in `OpenSearchPluginModule`

## Fixing PPL Bugs

Use `/ppl-bugfix #<issue_number>` to fix PPL bugs. It dispatches a subagent in an isolated worktree with a structured harness covering triage, fix, tests, and PR creation.

## Adding New PPL Commands

Follow the checklist in `docs/dev/ppl-commands.md`:
1. Update lexer/parser grammars (OpenSearchPPLLexer.g4, OpenSearchPPLParser.g4)
2. Add AST node under `org.opensearch.sql.ast.tree`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/opensearch-project) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
