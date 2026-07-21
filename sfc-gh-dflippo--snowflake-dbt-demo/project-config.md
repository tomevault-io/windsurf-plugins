---
trigger: always_on
description: Context and guidelines for AI coding agents working on this dbt + Snowflake data engineering project.
---

# AGENTS.md

Context and guidelines for AI coding agents working on this dbt + Snowflake data engineering project.

## Project Context

This is a **modern data engineering project** built with dbt-core and Snowflake, implementing industry best practices for analytics engineering.

---

## Technology Stack

### Core Technologies

- **dbt-core**: data transformation framework
- **dbt-snowflake**: Snowflake adapter for dbt
- **Snowflake**: Relational database
- **Streamlit in Snowflake**: Preferred graphical user interface (if needed)
- **Schemachange**: CI/CD for database objects outside dbt
- **Python**: 3.11+ with Snowpark for advanced analytics and ML models

**Version Compatibility**: dbt versions should align with [dbt Projects on Snowflake](https://docs.snowflake.com/en/user-guide/data-engineering/dbt-projects-on-snowflake#dbt-projects) requirements (dbt-core 1.9.4, dbt-snowflake 1.9.2)

### Key dbt Packages

- **dbt_constraints**: Database-level constraint enforcement (primary keys, foreign keys)
- **dbt_utils**: Utility macros and helper functions for common transformations
- **dbt_artifacts**: dbt logging to Snowflake database tables

### Development Tools

- **Snowflake CLI (`snow` command)**: Execution of database commands and scripts
- **dbt Projects on Snowflake**: Preferred execution environment when not developing locally
- **conda and uv**: Preferred Python package managers
- **schemachange**: Preferred CI/CD deployment tool for non-dbt objects (procedures, UDF, tasks, etc.)
- **Taskmaster AI**: Task-driven development workflow management
- **Git**: Version control with feature branch strategy
- **Playwright**: Testing GUI applications

---

## Agent Guidelines and Constraints

### Code Standards

- **Consistency** - Follow established patterns across the project
- **Testability** - Models should have appropriate data quality tests
- **Documentation** - Document business logic, complex transformations, models, and columns

### Legacy Rules (Cursor-specific)

For Cursor IDE integration, see `.cursor/rules/`:

- **[dbt.mdc](.cursor/rules/dbt.mdc)** - Complete dbt modeling guidelines
- **[dbt-observability.mdc](.cursor/rules/dbt-observability.mdc)** - Original observability guide
- **[snowflake-cli.mdc](.cursor/rules/snowflake-cli.mdc)** - Snowflake operations
- **[streamlit.mdc](.cursor/rules/streamlit.mdc)** - Streamlit development
- **[playwright.mdc](.cursor/rules/playwright.mdc)** - Browser testing
- **[schemachange.mdc](.cursor/rules/schemachange.mdc)** - Database migrations

---

### Safety and permissions

#### Allowed without prompt

- read files, list files
- reformat SQL
- executing dbt models
- query Snowflake database

#### Ask first

- Adding dbt macros, dbt packages, or python libraries
- git push
- deleting files, chmod

### Security Requirements

- **Never hardcode credentials** - Always use configuration files or environment variables

### Performance Guidelines

- **Use incremental materialization** for data pipelines with many rows
- **Apply appropriate clustering keys** for frequently queried columns
- **Size warehouses** based on execution time and model complexity

### Testing Requirements

- **Use `dbt build` instead of `dbt run`** to run dbt tests after model execution (`dbt build --select modelname`)
- **Use dbt_constraints** for primary/unique/foreign key validation

### Deployment Process

- **Test connection** with `dbt debug` before deployment
- **Deploy to dev, test, prod using Python script** with `python deploy_dbt_project.py --target environment_name`
- **Validate in production** with `dbt build --target prod`

---

## Specification-Driven Development Process

This project follows **Specification-Driven Development (SDD)** methodology, where a detailed, often executable, specification serves as the blueprint for development, testing, and documentation. This approach ensures stakeholder alignment, improves code quality, and enables efficient development cycles.

### Core SDD Principles

- **Early Specification Definition** - Clear requirements, objectives, and constraints upfront
- **Stakeholder Collaboration** - Engage all parties to align with business goals
- **Iterative Refinement** - Continuously update specifications based on feedback
- **Traceability** - Maintain clear links between specifications and implementation
- **Test-First Approach** - Define validation criteria before implementation

### Development Workflow

#### **Phase 1: Define the Functional and Technical Specifications**

1. **Gather Requirements** - Start with user stories or high-level requirements
   - Create comprehensive requirements, plans, and tasks in a PRD markdown document
   - Define functional requirements and design goals
   - Document business logic and transformation requirements
   - Specify acceptance criteria and validation requirements

2. **Write Clear Specs** - Document specifications in structured format
   - Use natural language that is easy for technology architects and engineers to review
   - Define data models, APIs, interfaces, and system behavior
   - Outline performance and scalability requirements
   - Establish quality and testing standards


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sfc-gh-dflippo/snowflake-dbt-demo](https://github.com/sfc-gh-dflippo/snowflake-dbt-demo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
