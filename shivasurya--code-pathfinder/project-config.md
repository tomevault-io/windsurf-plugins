---
trigger: always_on
description: GenerateSha256(exprType + node.Content(sourceCode))
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Essential Build Commands

### Building the Binary
```bash
cd sast-engine
gradle buildGo
```
The binary is output to `build/go/pathfinder`. The build automatically cleans previous builds first.

### Running Tests
```bash
gradle testGo          # Run all Go tests
go test ./...          # Direct Go test command
go test -v ./graph/... # Run tests for specific package with verbose output
```

### Linting
```bash
gradle lintGo
# or directly:
golangci-lint run
```

### Running the Binary
```bash
# MCP server mode (for AI coding assistants)
./build/go/pathfinder serve --project <path>
./build/go/pathfinder serve --http --address :8080 --project <path>

# Scan mode (using Python SDK rules)
./build/go/pathfinder scan --project <path> --ruleset <path_to_rules>

# CI mode (loads rules from remote/local, outputs SARIF/JSON/CSV)
./build/go/pathfinder ci --project <path> --ruleset cpf/java --output sarif

# Resolution diagnostics
./build/go/pathfinder resolution-report --project <path>

# Taint analysis diagnostics
./build/go/pathfinder diagnose --project <path>
```

### Running a Single Test
```bash
go test -v -run TestBuildCallGraph ./graph/callgraph/builder/
go test -v -run TestTypeInference ./graph/callgraph/resolution/
```

## High-Level Architecture

Code Pathfinder is a multi-stage analysis pipeline:

```
Source Files (.py, .java, Dockerfile, docker-compose.yml)
    ↓
Tree-Sitter AST Parsing (parallel workers)
    ↓
Code Graph (Functions, Classes, Call Edges)
    ↓
Type Inference Engine (bidirectional, return types, variable assignments)
    ↓
Call Graph Builder (5-pass algorithm)
    ↓
MCP Server / Python SDK Rules
    ↓
Output Formats (JSON, SARIF, CSV, Text)
```

### Core Packages

**sast-engine/graph/** - Code graph construction and management
- `initialize.go`: Multi-threaded file parsing with parallel workers
- `parser.go`: AST traversal orchestrator (language-agnostic entry point)
- `parser_java.go`: Java-specific node parsing
- `parser_python.go`: Python-specific node parsing
- `utils.go`: SHA256-based ID generation, file operations

**sast-engine/graph/callgraph/** - Call graph and type inference
- `builder/builder.go`: 5-pass call graph construction algorithm
- `resolution/inference.go`: Type inference engine with bidirectional inference
- `resolution/return_type.go`: Return type extraction from AST
- `extraction/variables.go`: Variable assignment type tracking
- `registry/attribute.go`: Class attribute registry
- `registry/builtin.go`: Python builtin types registry

**sast-engine/cmd/** - CLI interface
- `serve.go`: MCP server for AI coding assistants
- `scan.go`: Scan project against local ruleset
- `ci.go`: CI/CD integration with rule loading from codepathfinder.dev
- `resolution_report.go`: Call resolution diagnostics
- `diagnose.go`: Taint analysis diagnostics

**sast-engine/model/** - AST data models
- `stmt.go`: Statement models (if/while/for/blocks)
- `expr.go`: Expression models
- `location.go`: Source location tracking for lazy loading

**sast-engine/analytics/** - Optional PostHog telemetry

## Critical Design Patterns

### Node ID Generation
All node IDs are deterministic SHA256 hashes to ensure consistency across runs:
```go
// Methods: method:<name>-<params>-<file>:<line>:<col>
GenerateMethodID("method:methodName", []string{params}, filepath)

// Expressions: <type>+<content>
GenerateSha256(exprType + node.Content(sourceCode))
```

This enables:
- Consistent results despite multi-threaded parsing
- Deduplication of identical constructs
- Reliable linking between method invocations and declarations

### Lazy Loading with SourceLocation
Nodes store `StartByte` and `EndByte` offsets instead of full code snippets:
```go
type Node struct {
    SourceLocation *SourceLocation // File path + byte offsets
}

func (n *Node) GetCodeSnippet() string {
    content := readFile(n.SourceLocation.File)
    return string(content[StartByte:EndByte])
}
```

This reduces memory usage from ~2.32 GB to ~2.18 GB for large codebases (27k+ methods). Code snippets are read on-demand, leveraging OS page caching for performance.

### Cartesian Product Query Optimization
Multi-entity queries (e.g., "find method md calling method target") generate exhaustive combinations:
```go
// Single entity: O(n) linear filtering
// Two entities: O(n²) pairwise matching with early pruning

for _, lhsNode := range typeIndex[selectList[0].Entity] {
    for _, rhsNode := range typeIndex[selectList[1].Entity] {
        if FilterEntities([]*Node{lhsNode, rhsNode}, expression) {
            validPairs = append(validPairs, []*Node{lhsNode, rhsNode})
        }
    }
}
```

**Performance tip**: Limit multi-entity queries to related types (e.g., method + invocation) to avoid exponential explosion.

### Worker Pool Concurrency
File parsing uses 5 concurrent workers to balance parallelism with overhead:
```go
// In initialize.go
numWorkers := 5
for i := 0; i < numWorkers; i++ {
    go worker(i + 1)
}
```

Each worker has its own tree-sitter parser instance to avoid thread-safety issues.

### Object Pooling
Environment maps are pooled to reduce GC pressure during query evaluation:
```go
var envMapPool = sync.Pool{

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shivasurya/code-pathfinder](https://github.com/shivasurya/code-pathfinder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
