---
trigger: always_on
description: This repository hosts **Stock Indicators for .NET**, the production source for the <a href="https://www.nuget.org/packages/FacioQuo.Stock.Indicators">FacioQuo.Stock.Indicators</a> NuGet package. The library offers financial market technical analysis indicators with a focus on accuracy, performance, and ergonomics for financial analytics.
---

# Stock Indicators for .NET

This repository hosts **Stock Indicators for .NET**, the production source for the <a href="https://www.nuget.org/packages/FacioQuo.Stock.Indicators">FacioQuo.Stock.Indicators</a> NuGet package. The library offers financial market technical analysis indicators with a focus on accuracy, performance, and ergonomics for financial analytics.

- Multi-targets `net10.0`, `net9.0`, and `net8.0` with analyzers enabled for strict code quality.
- Active development expands streaming indicator support — see [src/Common/AGENTS.md](src/Common/AGENTS.md) for the framework guidance before modifying stateful pipelines (BufferList or StreamHub).
- Documentation at <a href="https://dotnet.stockindicators.dev">dotnet.stockindicators.dev</a> is sourced from the `docs/` content in this repository.

## Primary directive

Provide financial market developers with mathematically exact, high-performance technical analysis indicators they can depend on in production — so they can build trading and analytics applications with full confidence in numerical correctness.

## Secondary directives

1. Surface every indicator's behavior, limitations, and integration path through clear documentation and ergonomic APIs, so developers can discover and use any indicator without guesswork (not as important as primary)
2. Protect downstream users from silent numerical errors through comprehensive validation, deterministic warmup, and ≥ 98% test coverage — ensuring every deployment succeeds (not as important as #1)
3. Ensure all three indicator styles — Series, Buffer, Stream — produce identical results, so developers can switch between batch and real-time APIs without concern for numerical differences (not as important as #2)

## AI-first development model

This repository is optimized for AI agent contributions. Agents are the primary contributors; humans set direction, review, and validate outcomes.

- **Agents**: infer and proceed autonomously, generate complete implementations (code + tests + docs) in one pass, execute quality gates before yielding, document assumptions
- **Humans**: set strategic direction, make architectural decisions, review agent output, validate outcomes — not micromanage implementation
- **Voice**: imperative, present tense; no historical context; outcome-focused

## Guiding principles

This library follows six core principles that balance usability, performance, precision, and security: **Mathematical Precision** (non-negotiable), **Performance First** (critical), **Comprehensive Validation**, **Test-Driven Quality**, **Documentation Excellence**, and **Scope & Stewardship**.

See [PRINCIPLES.md](docs/PRINCIPLES.md) for constitutional philosophy and rationale. This file (AGENTS.md) provides operational implementation guidance.

## Repository layout

```plaintext
(root)
├── src/                   # Library source code
│    ├── Common/           # Shared utilities, base classes, and common types
│    ├── Indicators/       # Indicator implementations (a-b … t-z groups)
│    └── Indicators.csproj # Main project file
├── tests/                 # Unit, integration, performance, and simulation suites
├── tools/                 # Performance benchmarks, baselines, and analysis scripts
├── docs/                  # Public documentation site (VitePress)
└── Stock.Indicators.sln   # Primary solution for src + tests
```

## Commands

```bash
# Build
dotnet build "Stock.Indicators.sln" -v minimal --nologo

# Test (unit; integration tests run in CI only)
dotnet test "Stock.Indicators.sln" --no-restore --nologo

# Lint: .NET format
dotnet format --severity info --no-restore

# Lint: Markdown
npx markdownlint-cli2 --fix

# All quality gates (abbreviated)
dotnet format --no-restore && dotnet build && dotnet test --no-restore && npx markdownlint-cli2
```

See the code-completion skill for the complete quality gates checklist, Roslynator commands, and VS Code task equivalents.

## Skills for development

This repository uses Agent Skills (.agents/skills/) for domain-specific guidance. Skills are automatically loaded when relevant:

| Skill | Description | When to use |
| ----- | ----------- | ----------- |
| indicator-series | Series indicator development - mathematical precision, validation patterns, test coverage | Implementing new Series indicators, validating calculations, structuring tests |
| indicator-buffer | BufferList indicator development - incremental processing, interface selection, buffer management | Implementing BufferList indicators, choosing interfaces, managing state efficiently |
| indicator-stream | StreamHub indicator development - implementation patterns, provider selection, state management | Implementing StreamHub indicators, choosing provider base classes, optimizing real-time processing |
| indicator-catalog | Catalog entry creation and registration | Creating indicator catalog entries for automation |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [facioquo/stock-indicators-dotnet](https://github.com/facioquo/stock-indicators-dotnet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
