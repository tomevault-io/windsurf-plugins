---
trigger: always_on
description: Licensed to Julian Hyde under one or more contributor license
---

<!--
{% comment %}
Licensed to Julian Hyde under one or more contributor license
agreements.  See the NOTICE file distributed with this work
for additional information regarding copyright ownership.
Julian Hyde licenses this file to you under the Apache
License, Version 2.0 (the "License"); you may not use this
file except in compliance with the License.  You may obtain a
copy of the License at

http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing,
software distributed under the License is distributed on an
"AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND,
either express or implied.  See the License for the specific
language governing permissions and limitations under the
License.
{% endcomment %}
-->

# Claude Code Notes

This file provides guidance to Claude Code (claude.ai/code) when working
with code in this repository.

## Overview

Morel is a Standard ML interpreter with relational extensions,
implemented in Java. It allows users to write Standard ML code with
SQL-like query expressions to operate on in-memory data structures.
The project uses Apache Calcite for query optimization and planning.

## Build and Test Commands

### Building
```bash
./mvnw install              # Full build with all checks
./mvnw verify               # Compile and run tests
./mvnw compile              # Compile only
```

### Running Tests
```bash
./mvnw test                 # Run all tests
./mvnw test -Dtest=MainTest # Run specific test class
./mvnw test -Dtest=MainTest#testRepl # Run specific test method

# Run individual .smli script test files
./morel src/test/resources/script/wordle.smli

# Run individual script with visible output (for debugging)
# The --echo flag shows test output to stdout in real-time
./morel --echo src/test/resources/script/wordle.smli
```

### Running the Shell
```bash
./morel                     # Start interactive REPL
./morel -e '1 + 2'          # Evaluate expression and exit
```

### Code Quality
```bash
./mvnw checkstyle:check     # Run checkstyle
./mvnw javadoc:javadoc      # Generate javadoc
```

Note: The build uses Google Java Format automatically during the
`process-sources` phase. Checkstyle runs in the same phase.

## Architecture

Morel follows a traditional interpreter pipeline:
Parse → Type Check → Compile → Evaluate.

### Core Components

**Parser (`net.hydromatic.morel.parse`)**
- `MorelParser.jj`: JavaCC grammar for Standard ML plus extensions
- `MorelParserImpl`: Generated parser implementation
- Produces AST (`Ast` nodes)

**AST Layer (`net.hydromatic.morel.ast`)**
- `Ast`: User-facing abstract syntax tree from parser
- `Core`: Internal representation after type resolution
- `AstBuilder`, `CoreBuilder`: Fluent builders for constructing nodes
- `Visitor`, `Shuttle`: Tree traversal patterns

**Type System (`net.hydromatic.morel.type`)**
- `TypeSystem`: Central registry for types
- `Type` hierarchy: `PrimitiveType`, `RecordType`, `TupleType`,
  `ListType`, `FnType`, `DataType`, `TypeVar`, etc.
- `TypeVar`: Polymorphic type variables (parametric polymorphism)
- `TypeUnifier`: Hindley-Milner type inference using unification
- `Binding`: Associates names with types and values

**Compilation (`net.hydromatic.morel.compile`)**
- `TypeResolver`: Type inference and checking; converts `Ast` to `Core`
- `Compiler`: Compiles typed `Core` expressions into executable `Code`
- `Environment`: Symbol table holding bindings
- `BuiltIn`: Defines all built-in functions, operators, and types
- `CalciteCompiler`: Translates relational expressions to Calcite plans
- `Resolver`: Resolves names and converts patterns to code

**Evaluation (`net.hydromatic.morel.eval`)**
- `Code`: Interface for executable code nodes
- `Codes`: Implementations of all code types
- `EvalEnv`: Runtime environment mapping variables to values
- `Closure`: Function values that capture their environment
- `Applicable`: Function objects with apply methods
- `Session`: Maintains REPL state and configuration

**Datalog (`net.hydromatic.morel.datalog`)**
- `DatalogParserImpl`: JavaCC parser for Datalog syntax
- `DatalogAst`: Datalog abstract syntax tree nodes
- `DatalogAnalyzer`: Safety and stratification checking
- `DatalogTranslator`: Translates Datalog to Morel source
- `DatalogEvaluator`: Orchestrates parse → analyze → translate → execute

**Foreign Interface (`net.hydromatic.morel.foreign`)**
- `ForeignValue`: Interface for exposing Java values/functions to Morel
- `Calcite`: Integration with Apache Calcite for relational queries
- `DataSet`: Abstraction for queryable datasets (backed by Calcite)

**Main Entry Points**
- `Main`: REPL implementation with shell and sub-shell support
- `Shell`: Handles command execution and error reporting

### Key Execution Flow

1. **Parsing**: User input → `MorelParser` → `Ast` nodes
2. **Type Resolution**: `Ast` + `Environment` → `TypeResolver` →
   typed `Core` nodes
3. **Compilation**: `Core` → `Compiler` → `Code` nodes
4. **Evaluation**: `Code` + `EvalEnv` → execution → result value

### Important Implementation Details

**Type Inference**
- Uses Hindley-Milner algorithm (Algorithm W) via `TypeResolver`
- Type variables represent unknown types during inference
- Unification (`TypeUnifier`) propagates type constraints

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hydromatic/morel](https://github.com/hydromatic/morel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
