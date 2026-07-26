---
trigger: always_on
description: **RepoSense** is a contribution analysis tool for Git repositories that generates interactive visualizations of programmer activities across single or multiple repositories. It's designed for educators and managers to track the work done in projects and analyze code contributions chronologically.
---

# GitHub Copilot Instructions for RepoSense

## Project Overview

**RepoSense** is a contribution analysis tool for Git repositories that generates interactive visualizations of programmer activities across single or multiple repositories. It's designed for educators and managers to track the work done in projects and analyze code contributions chronologically.

**Key Purpose:**

- Analyze Git repository contribution patterns
- Visualize programmer activities across time
- Generate interactive HTML reports with commit and authorship insights
- Support configuration-driven analysis for multiple repositories

## Technology Stack

### Backend

- **Language:** Java 11
- **Build System:** Gradle
- **Key Dependencies:**
  - Gson 2.9.0 (JSON serialization/deserialization)
  - JLHttp 2.6 (HTTP server)
  - Argparse4j 0.9.0 (CLI argument parsing)
  - Apache Commons CSV 1.9.0 (CSV parsing)
  - Jackson DataFormat YAML 2.17.0 (YAML parsing)
  - Jansi 2.4.1 (colored console output)
  - JUnit 5.8.2+ (testing)

### Frontend

- **Language:** TypeScript with Vue 3
- **Build Tool:** Vite
- **Package Manager:** npm
- **Key Dependencies:**
  - Vue 3
  - Vue Router
  - Vuex (State management)
  - Zod (Runtime schema validation for JSON report files)
  - ESLint and Stylelint (Linting)
  - Cypress (E2E testing)
  - Highlight.js (Code highlighting)
  - Markdown-it (Markdown parsing)
  - FontAwesome (Icons)
  - JSZip (ZIP archive handling)
  - Minimatch (Glob pattern matching)

### DevOps & Documentation

- **CI/CD:** GitHub Actions
- **Documentation:** MarkBind
- **Code Coverage:** Codecov
- **Deployment:** Netlify and Surge

## Project Structure

```
RepoSense/
├── src/
│   ├── main/java/reposense/          # Main backend source code
│   │   ├── parser/                    # CLI, CSV, JSON parsing
│   │   ├── git/                       # Git command wrappers
│   │   ├── commits/                   # Commit analysis
│   │   ├── authorship/                # File authorship analysis
│   │   ├── report/                    # Report generation
│   │   ├── system/                    # OS/system interactions
│   │   ├── model/                     # Data structures
│   │   └── RepoSense.java             # Main entry point
│   ├── test/java/reposense/           # Unit tests
│   └── systemtest/java/reposense/     # System tests
├── frontend/
│   ├── src/
│   │   ├── components/                # Vue components
│   │   ├── views/                     # Page-level components
│   │   ├── router/                    # Vue Router configuration
│   │   ├── store/                     # Vuex store
│   │   ├── types/                     # TypeScript types (includes Zod schemas)
│   │   ├── utils/                     # Utility functions
│   │   ├── mixin/                     # Vue mixins for shared logic
│   │   └── main.ts                    # Entry point
│   ├── config-wizard/                 # Standalone YAML configuration wizard app
│   ├── cypress/                       # E2E tests
│   └── package.json
├── docs/
│   ├── dg/                            # Developer guide
│   ├── ug/                            # User guide
│   └── _markbind/                     # MarkBind configuration
├── config/
│   ├── author-config.csv              # Author configuration template
│   ├── repo-config.csv                # Repository configuration template
│   ├── group-config.csv               # Group configuration template
│   ├── report-config.yaml             # Report metadata
│   ├── author-blurbs.md               # Per-author blurb descriptions
│   ├── repo-blurbs.md                 # Per-repo blurb descriptions
│   ├── chart-blurbs.md                # Per-chart blurb descriptions
│   ├── checks/                        # CI check scripts
│   ├── checkstyle/                    # Checkstyle configuration
│   └── gh-actions/                    # GitHub Actions helper scripts
├── build.gradle                       # Gradle build configuration
└── .github/                           # GitHub workflows and templates
```

## Architecture Overview

RepoSense follows a modular architecture with clear separation of concerns:

### Backend Pipeline

1. **Parser** (`parser/` package)

   - `ArgsParser`: Parses CLI arguments into `CliArguments` object
   - `CsvParser`: Abstract parser for CSV configuration files (author, group, repo configs)
   - `JsonParser`: Abstract parser for JSON configuration files
   - `YamlParser`: Abstract parser for YAML configuration files
   - `StandaloneConfigJsonParser`: Parses `_reposense/config.json`
   - `ReportConfigYamlParser`: Parses `report-config.yaml`
   - `BlurbMarkdownParser` and subtypes (`AuthorBlurbMarkdownParser`, `RepoBlurbMarkdownParser`, `ChartBlurbMarkdownParser`): Parse blurb Markdown files

2. **Git Layer** (`git/` package)

   - Wrapper classes for Git commands (blame, log, clone, branch, etc.)
   - `GitUtil`: Helper functions for Git operations
   - `GitVersion`: Ensures compatible Git version

3. **Analysis Layer**

   - **`CommitsReporter`**: Analyzes commit history

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [reposense/RepoSense](https://github.com/reposense/RepoSense) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
