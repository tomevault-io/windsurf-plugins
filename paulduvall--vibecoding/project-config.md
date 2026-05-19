---
trigger: always_on
description: This file provides Claude Code with essential project context, rules, and implementation details. This is derived from the Windsurf-specific `.windsurfrules.md` but adapted for Claude Code workflow.
---

# Project Context for Claude Code: CLAUDE.md

This file provides Claude Code with essential project context, rules, and implementation details. This is derived from the Windsurf-specific `.windsurfrules.md` but adapted for Claude Code workflow.

## Project Overview

**Vibe Coding Digest** is an automated content aggregation and summarization tool that delivers curated, daily email digests of the most relevant developments in AI, developer tools, and emerging technology.

### Key Components
- **RSS Feed Aggregation**: Multi-source content collection with concurrent processing from 29+ feeds
- **External Configuration**: JSON/YAML feed management with categorization and enable/disable controls
- **AI Summarization**: OpenAI-powered content summarization with error handling
- **Email Delivery**: SendGrid integration for HTML digest delivery
- **AWS Blog Search**: Targeted search functionality for AWS-specific content
- **ATDD Test Suite**: Comprehensive unit, integration, and acceptance testing with user story traceability

## Project Structure

```
vibecoding/
├── src/                    # Main source package
│   ├── __init__.py        # Package exports and metadata
│   ├── vibe_digest.py     # Main orchestration and entry point
│   ├── models.py          # Data models (DigestItem)
│   ├── feeds.py           # RSS feed management and fetching
│   ├── config_loader.py   # External configuration management
│   ├── summarize.py       # OpenAI summarization logic
│   ├── email_utils.py     # SendGrid email functionality
│   └── aws_blog_search.py # AWS blog search functionality
├── tests/                 # Comprehensive test suite
│   ├── test_vibe_digest.py     # Core functionality tests
│   ├── test_aws_blog_search.py # AWS search tests
│   ├── test_integration.py     # End-to-end pipeline tests
│   ├── test_optimizations.py   # Performance optimization tests
│   └── features/              # ATDD feature specifications
│       ├── digest_workflow.feature      # Main digest scenarios
│       ├── externalized_config.feature  # Configuration scenarios
│       ├── environment.py               # Behave setup
│       └── steps/                       # Step definitions
│           ├── digest_steps.py          # Main workflow steps
│           └── config_steps.py          # Configuration steps
├── docs/                  # Project documentation
│   ├── user_stories.md    # User stories with acceptance criteria
│   ├── traceability_matrix.md # ATDD traceability documentation
│   └── prd.md            # Product Requirements Document
├── .github/
│   └── workflows/
│       └── vibe-coding-digest.yml # CI/CD automation
├── feeds_config.json      # Default external feed configuration
├── .attdrules.md         # ATDD/BDD development guidelines
├── pyproject.toml         # Modern Python package configuration
├── requirements.txt       # Runtime dependencies (includes PyYAML)
└── run.sh                # Development utility script
```

## Development Standards

### Python Environment
- **Required Version**: Python 3.11 (enforced via `run.sh`)
- **Virtual Environment**: All operations use `.venv` managed by `run.sh`
- **Package Management**: Uses modern `pyproject.toml` configuration
- **Import Strategy**: Absolute imports (`from src.module import ...`)

### Code Quality Requirements
- **Type Hints**: Full type annotation coverage required
- **Linting**: Flake8 compliance enforced (PEP8)
- **Testing**: Pytest with coverage reporting
- **Documentation**: Comprehensive docstrings and comments

### Development Rules & Guidelines
The project follows comprehensive development standards defined in specialized rule files:

- **CI/CD Practices**: [`.cicdrules.md`](.cicdrules.md) - Complete CI/CD best practices framework with 95 rules covering build quality, security, automation, and delivery excellence
- **ATDD/BDD Development**: [`.attdrules.md`](.attdrules.md) - Acceptance Test-Driven Development methodology for AI-assisted development with executable specifications
- **Code Quality & Refactoring**: [`.refactoringrules.md`](.refactoringrules.md) - Comprehensive catalog of code smells and refactoring techniques based on Martin Fowler's catalog
- **AWS Security**: [`.awssecurityrules.md`](.awssecurityrules.md) - MECE framework for AWS security best practices covering IAM, infrastructure protection, data security, and compliance
- **IAM Role Management**: [`.iamrolerules.md`](.iamrolerules.md) - Guidelines for creating, refining, and maintaining AWS IAM roles and policies with least-privilege principles

All contributors must follow these specialized rules in addition to the standards outlined in this document.

### Testing Standards
- **ATDD-Driven Development**: Write Gherkin feature specifications before implementation (see [`.attdrules.md`](.attdrules.md))
- **Test Coverage**: Pytest-cov integration with HTML reports  
- **Test Types**: Unit, integration, and acceptance tests with user story traceability
- **Mock Strategy**: Comprehensive external service mocking
- **User Story Mapping**: Each test maps to specific user stories (US-001 through US-305)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PaulDuvall/vibecoding](https://github.com/PaulDuvall/vibecoding) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
