---
trigger: always_on
description: Debtmap is a code complexity and technical debt analyzer written in Rust. It analyzes Rust, Python, JavaScript, and TypeScript codebases to identify complexity patterns, technical debt, and architectural issues.
---

# Debtmap Development Guidelines

## Project Context

Debtmap is a code complexity and technical debt analyzer written in Rust. It analyzes Rust, Python, JavaScript, and TypeScript codebases to identify complexity patterns, technical debt, and architectural issues.

## Philosophy

### Core Beliefs

- **Functional-first design** - Pure functions with minimal side effects
- **Immutable data structures** - Use `im` crate for persistent collections
- **Composable analysis pipelines** - Chain pure transformations
- **Incremental progress over big bangs** - Small, testable changes
- **Type-driven development** - Let the type system guide correctness
- **Performance through parallelism** - Use `rayon` for data parallelism

### Rust-Specific Principles

#### Functional Programming in Rust

- **Prefer iterators over loops** - Use `.map()`, `.filter()`, `.fold()` for transformations
- **Leverage trait composition** - Build complex behavior through trait bounds
- **Minimize mutable state** - Use `Cow<'_, T>` or immutable builders
- **Pure computation cores** - Separate parsing, analysis, and I/O concerns
- **Result chains** - Use `?` operator and monadic patterns for error handling

#### Memory and Performance

- **Zero-copy where possible** - Use string slices and borrowed data
- **Parallel processing** - Use `rayon::par_iter()` for CPU-intensive analysis
- **Lazy evaluation** - Compute only what's needed when needed
- **Efficient collections** - Use `dashmap::DashMap` for concurrent access

#### Type System Leverage

- **Phantom types** - Encode state transitions in types
- **Builder patterns** - Ensure correct construction through types
- **Exhaustive pattern matching** - Handle all enum variants explicitly
- **Newtype wrappers** - Prevent mixing incompatible values

## Architecture Patterns

### Module Organization

```
src/
├── core/           # Core data types and utilities
├── analyzers/      # Language-specific parsers and analyzers
├── analysis/       # Advanced analysis algorithms
├── debt/           # Technical debt detection patterns
├── complexity/     # Complexity calculation algorithms
├── risk/           # Risk assessment and prioritization
├── io/             # Input/output formatting and handling
└── testing/        # Test analysis and quality metrics
```

### Data Flow Architecture

1. **Parser Layer** - Language-specific AST generation (pure)
2. **Analysis Layer** - Metric computation and pattern detection (pure)
3. **Aggregation Layer** - Combine results across files (functional)
4. **Output Layer** - Format and emit results (I/O boundary)

### Functional Composition Patterns

```rust
// Good: Functional pipeline
fn analyze_project(config: &Config) -> Result<AnalysisResults> {
    discover_files(config)?
        .into_par_iter()
        .map(|path| parse_file(&path))
        .collect::<Result<Vec<_>>>()?
        .into_iter()
        .map(|ast| analyze_complexity(&ast))
        .fold(AnalysisResults::empty(), |acc, result| acc.merge(result))
}

// Avoid: Imperative with mutation
fn analyze_project_bad(config: &Config) -> Result<AnalysisResults> {
    let mut results = AnalysisResults::new();
    for file in discover_files(config)? {
        let ast = parse_file(&file)?;
        let metrics = analyze_complexity(&ast);
        results.add_metrics(metrics); // Mutation!
    }
    Ok(results)
}
```

## Function Design Guidelines

### Size and Complexity Limits

- **Maximum function length**: 20 lines (prefer 5-10)
- **Maximum cyclomatic complexity**: 5
- **Maximum nesting depth**: 2 levels
- **Maximum parameters**: 3-4 (use structs for more)

### Pure Function Patterns

```rust
// Good: Pure function with clear input/output
fn calculate_cognitive_complexity(ast: &Ast) -> u32 {
    ast.functions()
        .map(|func| func.cognitive_weight())
        .sum()
}

// Good: Pure transformation
fn normalize_path(path: &Path) -> PathBuf {
    path.components()
        .filter(|c| !matches!(c, Component::CurDir))
        .collect()
}

// Avoid: Side effects mixed with computation
fn calculate_complexity_bad(ast: &Ast) -> u32 {
    println!("Calculating complexity..."); // Side effect!
    let mut total = 0;
    for func in ast.functions() {
        total += func.cognitive_weight(); // Mutation!
    }
    total
}
```

### Error Handling Patterns

```rust
use anyhow::{Context, Result};

// Good: Functional error handling
fn parse_and_analyze(content: &str, path: &Path) -> Result<FileMetrics> {
    parse_content(content, path)
        .context("Failed to parse file")?
        .analyze()
        .context("Analysis failed")
}

// Good: Monadic chaining with custom extension
fn process_file(path: &Path) -> Result<AnalysisResult> {
    read_file(path)?
        .map_ok(|content| preprocess_content(&content))
        .and_then_async(|content| parse_content(&content, path))
        .map_ok(|ast| analyze_ast(&ast))
}
```

## Testing Standards

### Test Organization

- **Unit tests**: In same file as implementation using `#[cfg(test)]`
- **Integration tests**: In `tests/` directory for cross-module scenarios

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iepathos/debtmap](https://github.com/iepathos/debtmap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
