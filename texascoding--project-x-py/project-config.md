---
trigger: always_on
description: Use when:
---

# AI Agents for Development

The ProjectX Python SDK leverages specialized AI agents for development tasks. This guide explains how to work with these agents effectively for SDK development, maintenance, and enhancement.

## Agent Overview

The project uses multiple specialized AI agents, each optimized for specific development tasks:

- **python-developer**: Core SDK development and feature implementation
- **code-standards-enforcer**: Code quality, standards compliance, and IDE diagnostics
- **code-refactor**: Architecture improvements and code modernization
- **code-documenter**: Documentation creation and maintenance
- **code-debugger**: Issue diagnosis and troubleshooting
- **code-reviewer**: Code review and quality assurance

## Agent Selection Guidelines

### When to Use Each Agent

#### **python-developer**
**Best for**: Core SDK development and feature implementation

Use when:
- Implementing new trading components (OrderManager, PositionManager, etc.)
- Creating financial indicators with Polars DataFrames
- Building real-time data processing and WebSocket connections
- Developing new TradingSuite features
- Ensuring 100% async architecture compliance
- Handling Decimal price precision requirements

Example scenarios:
```
"Implement a new technical indicator for options flow analysis"
"Add WebSocket reconnection logic with exponential backoff"
"Create async order placement methods with bracket order support"
"Build a new risk management component with portfolio-level controls"
```

#### **code-standards-enforcer**
**Best for**: Maintaining code quality and standards

**CRITICAL**: Always check IDE diagnostics first with `mcp__ide__getDiagnostics`

Use when:
- **Before committing any changes** (proactive quality control)
- Preparing for pull requests
- Release validation processes
- Verifying 100% async architecture compliance
- Checking TradingSuite patterns adherence
- Ensuring Polars-only DataFrame usage
- Validating deprecation compliance
- Type safety verification with TypedDict/Protocol

Workflow:
1. **First**: Check `mcp__ide__getDiagnostics` for modified files
2. **Fix** any IDE diagnostic errors/warnings
3. **Then**: Run traditional linting tools (ruff, mypy)
4. **Verify** with IDE diagnostics again after fixes

Example scenarios:
```
"Check code quality before committing new indicator implementation"
"Validate async patterns in the new order management system"
"Ensure type safety compliance across all modified files"
"Verify deprecation warnings are properly implemented"
```

#### **code-refactor**
**Best for**: Architecture improvements and modernization

Use when:
- Migrating to TradingSuite patterns
- Optimizing Polars DataFrame operations
- Consolidating WebSocket handling
- Modernizing async patterns and removing legacy sync code
- Transitioning from monolithic to modular architectures
- Optimizing event system performance
- Implementing memory management improvements

Example scenarios:
```
"Refactor OrderManager to use EventBus for better decoupling"
"Optimize DataFrame operations in indicators for better performance"
"Migrate legacy synchronous code to modern async patterns"
"Consolidate WebSocket connections for improved resource usage"
```

#### **code-documenter**
**Best for**: Documentation creation and maintenance

Use when:
- Documenting new TradingSuite APIs and features
- Writing indicator function documentation with usage examples
- Explaining WebSocket events and data flow patterns
- Creating migration guides for breaking changes
- Maintaining README files and examples directory
- Writing deprecation notices with clear upgrade paths
- Updating docstrings with comprehensive type hints
- Creating tutorial notebooks and interactive examples

Example scenarios:
```
"Document the new risk management system with usage examples"
"Create migration guide for v3 to v4 breaking changes"
"Update API documentation for the enhanced order management system"
"Write comprehensive examples for the new indicator framework"
```

#### **code-debugger**
**Best for**: Issue diagnosis and troubleshooting

Use when:
- Investigating WebSocket disconnection issues
- Diagnosing order lifecycle failures
- Troubleshooting real-time data gaps
- Resolving event system deadlocks
- Fixing price precision errors
- Identifying memory leaks and performance bottlenecks
- Debugging AsyncIO-related issues
- Tracing SignalR connection problems

Example scenarios:
```
"Debug why orders aren't filling after placement"
"Investigate WebSocket reconnection failures under load"
"Trace event propagation issues in the real-time system"
"Diagnose memory leaks in long-running data streams"
```

#### **code-reviewer**
**Best for**: Code review and quality assurance

Use when:
- Reviewing async patterns and implementations
- Checking real-time performance characteristics
- Validating financial data integrity and precision
- Ensuring API stability and backward compatibility
- Conducting pre-release code reviews
- Reviewing pull requests for quality and standards

Example scenarios:
```
"Review the new bracket order implementation for correctness"
"Check real-time data processing for performance issues"
"Validate the new indicator calculations for accuracy"
"Review API changes for backward compatibility"
```

## Agent Command Requirements


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TexasCoding/project-x-py](https://github.com/TexasCoding/project-x-py) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
