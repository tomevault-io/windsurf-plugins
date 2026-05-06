---
trigger: always_on
description: AI coding agent instructions for the SQM codebase.
---

# SQM - Structured Query Model for Java

AI coding agent instructions for the SQM codebase.

## Project Overview

SQM is a Java 21+ SQL framework centered on a typed immutable AST.

Core capabilities include:

- parse SQL text into a structured model
- transform and rewrite the model
- validate statements against dialect and schema rules
- render model nodes back to SQL
- serialize SQL models as JSON
- build statements through a fluent DSL
- run middleware policy and decision pipelines

SQM is not just a parser/renderer pair. It is a multi-stage SQL pipeline with shared model semantics across modules.

## Architecture Overview

SQM uses a multi-module Maven layout with clear responsibility boundaries.

### Core Model

- `sqm-core` - immutable AST nodes, DSL helpers, visitor/transformer infrastructure, match API
- `sqm-json` - Jackson mixins for model serialization

### Parsing

- `sqm-parser` - parser SPI and shared parsing infrastructure
- `sqm-parser-ansi` - ANSI parser
- `sqm-parser-postgresql` - PostgreSQL parser extensions
- `sqm-parser-mysql` - MySQL parser extensions

### Rendering

- `sqm-render` - renderer SPI and shared rendering infrastructure
- `sqm-render-ansi` - ANSI renderers
- `sqm-render-postgresql` - PostgreSQL renderers
- `sqm-render-mysql` - MySQL renderers

### Validation

- `sqm-validate` - shared validation framework
- `sqm-validate-postgresql` - PostgreSQL validation dialect
- `sqm-validate-mysql` - MySQL validation dialect

### Catalog and Codegen

- `sqm-catalog` and dialect-specific catalog modules
- `sqm-codegen` and `sqm-codegen-maven-plugin`

### Control and Middleware

- `sqm-control` - rewrite, decision, and execution pipeline abstractions
- `sqm-middleware-*` - runtime hosts and integrations

### Integration and Examples

- `sqm-it` - integration tests
- `examples` - user-facing examples

### Planned Next Direction

- `sqm-transpile` is the next planned foundation module for source-to-target SQL transpilation.

## Model Conventions

### Immutable Node Model

AST nodes are immutable.

- never mutate node state
- use factory methods and builder-like copy methods
- transformers must return the same instance when unchanged and a new instance when changed

### Interface-First Public API

Public model types live in `io.sqm.core.*` as interfaces and records.

Current code does not consistently follow an old “all impls live in `io.sqm.core.internal`” pattern. Many node types use nested `record Impl(...)` implementations inside the public interface. Follow the pattern already used by the file you are changing rather than forcing a different implementation style.

### Visitors, Transformers, Matchers, JSON

When adding a new node type, the work is not complete until the node is integrated with:

- visitor dispatch
- recursive visitor behavior where needed
- recursive transformer behavior
- match API coverage
- JSON mixin coverage
- documentation in the model docs

### Semantic Core vs Dialect Syntax

Prefer semantic nodes in `sqm-core` when the concept is portable across dialects.

Examples already moving in this direction include:

- `CastExpr`
- `RegexPredicate`
- `AtTimeZoneExpr`

The current transpilation direction also treats `ConcatExpr` as the first-wave example of a semantic node that should unify PostgreSQL `||` and MySQL `CONCAT(...)`.

Keep syntax-only or vendor-specific behavior out of the core model unless there is a clear reusable semantic abstraction.

## Parser and Renderer Guidance

### Parsers

Parsers should:

- accept syntax supported by their dialect
- produce the shared core model where semantics are portable
- produce dialect-specific nodes only when the concept is truly dialect-specific

### Renderers

Renderers should:

- render only syntax supported by the target dialect
- reject unsupported constructs rather than silently rewriting them
- stay focused on syntax emission, not semantic transpilation

### Transpilation

Cross-dialect semantic conversion belongs in the planned transpilation layer, not in ad hoc renderer fallbacks.

Portable concepts may still be promoted into `sqm-core` so that parsers and renderers meet at a shared semantic node and fewer pairwise transpile rules are needed.

## Testing Guidance

### Unit Tests

Prefer focused unit tests close to the module being changed.

- `sqm-core` - node behavior, matchers, visitors, transformers
- `sqm-parser-*` - parser coverage for valid, invalid, and boundary syntax
- `sqm-render-*` - renderer output and unsupported-feature rejection
- `sqm-validate*` - validation behavior and diagnostics
- `sqm-control` - rewrite and decision pipeline behavior

### Integration Tests

Use `sqm-it` for cross-module flows and round-trip coverage.

Typical integration shape:

1. parse SQL
2. inspect or transform model
3. validate if relevant
4. render SQL
5. compare normalized SQL where formatting is not the subject under test

### Important Testing Practices

- normalize whitespace in SQL assertions unless formatting is the behavior under test
- test both happy paths and failure paths
- add dialect-specific unsupported tests when a feature must be rejected
- use DSL helpers in tests when they make intent clearer than low-level node construction

## Common Development Patterns

### Adding a New Core Node


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [icher-g/sqm](https://github.com/icher-g/sqm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
