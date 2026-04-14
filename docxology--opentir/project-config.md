---
trigger: always_on
description: This is a sophisticated analysis framework for Palantir's open source ecosystem that combines statistical analysis, visualization, and comprehensive documentation generation.
---

# OpenTIR - Comprehensive Palantir OSS Analysis Framework
# Cursor IDE Rules and Context

## Project Overview
This is a sophisticated analysis framework for Palantir's open source ecosystem that combines statistical analysis, visualization, and comprehensive documentation generation.

## Architecture Philosophy
- **@/src**: Core connective tissue - modular, composable analysis components
- **@/examples**: Integration layer - clear control flow showcasing src/ method composition
- **@/multi_repo_analysis**: Generated comprehensive analysis outputs
- **@/repos**: Local repository storage for analysis

## Core Principles
1. **Modular Design**: Each src/ module is a standalone component with clear interfaces
2. **Composable Architecture**: examples/ demonstrate how src/ components integrate
3. **Comprehensive Analysis**: Statistical rigor with enterprise-grade insights
4. **Professional Documentation**: Mermaid diagrams, technical specs, and clear control flows
5. **Performance Optimization**: Intelligent caching and parallel processing

## Code Style & Standards

### Python Standards
- Use type hints for all functions and methods
- Follow PEP 8 with 88-character line length
- Use dataclasses for structured data
- Implement comprehensive docstrings with Args/Returns/Raises
- Use pathlib.Path for all file operations
- Implement proper error handling and logging

### Documentation Standards
- Every public method must have comprehensive docstrings
- Include parameter types, return types, and examples
- Use Mermaid diagrams for complex workflows
- Generate both technical and executive-level reports

### Analysis Standards
- Use scipy.stats for statistical tests (ANOVA, t-tests, chi-square)
- Use sklearn for PCA, clustering, and ML analysis
- Use matplotlib/seaborn/plotly for comprehensive visualizations
- Include statistical significance testing and confidence intervals

## Directory Structure

```
opentir/
├── src/                           # Core analysis components (connective tissue)
│   ├── code_analyzer.py          # Language-specific code parsing and analysis
│   ├── multi_repo_analyzer.py    # Cross-repository pattern analysis
│   ├── github_client.py          # GitHub API integration
│   ├── docs_generator.py         # Technical documentation generation
│   ├── repo_manager.py           # Repository management and caching
│   ├── utils.py                  # Shared utilities and logging
│   ├── config.py                 # Configuration management
│   ├── cli.py                    # Command-line interface
│   └── main.py                   # Main orchestrator class
├── examples/                      # Integration and control flow examples
│   ├── basic_usage.py            # Simple analysis workflow
│   ├── advanced_statistical_analysis.py  # Complex statistical analysis
│   └── cli_usage.md              # Command-line usage examples
├── multi_repo_analysis/          # Generated analysis outputs
│   ├── reports/                  # Generated markdown reports
│   ├── visualizations/           # Charts and graphs
│   ├── individual_analyses/      # Per-repository analysis
│   └── csv_exports/             # Machine-readable data
├── tests/                        # Comprehensive test suite
├── docs/                         # Project documentation
├── main.py                       # Top-level entry point
└── run.sh                        # Shell script orchestrator
```

## Component Responsibilities

### src/code_analyzer.py
- **Purpose**: Language-specific parsing and code element extraction
- **Key Methods**: analyze_repository(), _parse_functions(), _calculate_complexity()
- **Integration**: Used by multi_repo_analyzer for individual repo analysis

### src/multi_repo_analyzer.py
- **Purpose**: Cross-repository pattern analysis and aggregation
- **Key Methods**: analyze_all_repositories(), _extract_functionality_patterns()
- **Integration**: Orchestrates code_analyzer across multiple repositories

### src/github_client.py
- **Purpose**: GitHub API integration for repository discovery and metadata
- **Key Methods**: get_repositories(), fetch_repository_metadata()
- **Integration**: Feeds repository lists to repo_manager

### examples/advanced_statistical_analysis.py
- **Purpose**: Demonstrates comprehensive statistical analysis workflows
- **Key Features**: ANOVA, PCA, clustering, advanced visualizations
- **Integration**: Consumes src/ components to perform enterprise-grade analysis

## Development Guidelines

### When Adding New Features
1. **Start in src/**: Create modular, testable components
2. **Document thoroughly**: Include docstrings, type hints, and examples
3. **Add tests**: Ensure comprehensive test coverage
4. **Create examples**: Show integration in examples/ directory
5. **Update documentation**: Include in technical docs and README

### Statistical Analysis Standards
- Always include statistical significance testing
- Use appropriate statistical tests for data types
- Include confidence intervals and effect sizes
- Validate assumptions (normality, homoscedasticity, etc.)
- Handle missing data appropriately

### Visualization Standards
- Use consistent color palettes and styling

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/docxology) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
