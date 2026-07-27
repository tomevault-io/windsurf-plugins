---
trigger: always_on
description: This document provides guidance for GitHub Copilot when working on the Whyis project.
---

# GitHub Copilot Instructions for Whyis

This document provides guidance for GitHub Copilot when working on the Whyis project.

## Project Overview

Whyis is a nano-scale knowledge graph publishing, management, and analysis framework built with Python and Flask. It manages knowledge as nanopublications, which are the smallest publishable units of knowledge graphs with associated provenance and publication information.

### Key Technologies

- **Backend**: Python 3.8+ with Flask web framework (CI tests 3.8-3.11)
- **Knowledge Graph**: RDF, OWL, SPARQL, rdflib
- **Database**: Apache Jena Fuseki (included in setup)
- **Frontend**: Vue.js components, JavaScript
- **Task Queue**: Celery with Redis backend
- **Testing**: pytest, pytest-flask, pytest-cov
- **Linting**: pylint

## Repository Structure

```
whyis/
├── whyis/                  # Main Python package
│   ├── autonomic/         # Autonomous agents (FRIR, SETLR, etc.)
│   ├── blueprint/         # Flask blueprints
│   ├── commands/          # CLI commands
│   ├── config/            # Configuration modules
│   ├── database/          # Database utilities
│   ├── datastore/         # Data storage
│   ├── fuseki/            # Fuseki server integration
│   ├── importer/          # Data importers
│   ├── nanopub/           # Nanopublication handling
│   ├── plugins/           # Plugin system
│   ├── static/            # Frontend assets (JS, CSS, Vue components)
│   └── templates/         # Jinja2 templates
├── tests/                 # Test suite
│   ├── unit/             # Unit tests
│   ├── api/              # API endpoint tests
│   └── integration/      # Integration tests
├── docs/                  # Documentation
└── examples/              # Example applications
```

## Development Workflow

### Setting Up Development Environment

```bash
# Install development dependencies
pip install -r requirements-test.txt

# Install package in editable mode
pip install -e .

# Build frontend assets
cd whyis/static
npm install
npm run build-dev
```

### Running Tests

```bash
# Run all tests
pytest

# Run unit tests only
pytest tests/unit/

# Run with coverage
pytest --cov=whyis --cov-report=html --cov-report=term

# Run specific test file
pytest tests/unit/test_namespace.py

# Run tests in verbose mode
pytest -v

# Run tests and stop at first failure
pytest -x
```

### Code Quality

```bash
# Run pylint
pylint whyis/

# Check specific module
pylint whyis/namespace.py
```

## Coding Conventions

### Python Style

- **Follow PEP 8** for Python code style
- **Python 3.8+ syntax**: Use type hints where appropriate
- **Docstrings**: Use clear docstrings for modules, classes, and functions
- **Imports**: Group imports (standard library, third-party, local) with blank lines between groups
- **Line length**: Keep lines under 100 characters when possible

### Testing Conventions

- **Test files**: Use `test_*.py` naming convention
- **Test classes**: Start with `Test` prefix (e.g., `TestNamespace`)
- **Test functions**: Start with `test_` prefix
- **Arrange-Act-Assert pattern**: Structure tests clearly:
  ```python
  def test_function():
      # Arrange - set up test data
      input_data = prepare_data()
      
      # Act - execute the function being tested
      result = function_to_test(input_data)
      
      # Assert - verify the results
      assert result == expected_value
  ```
- **Use fixtures**: Define reusable test fixtures in `conftest.py`
- **Test coverage goal**: Aim for >80% coverage on new code
- **Docstrings in tests**: Explain what is being tested

### RDF and Semantic Web Conventions

- **Use rdflib**: For RDF graph operations
- **Namespaces**: Define namespaces in `whyis.namespace` module
- **SPARQL queries**: Use parameterized queries to prevent injection
- **URIs**: Follow best practices for URI design

### Flask Conventions

- **Blueprints**: Organize routes using Flask blueprints (but prefer view infrastructure over new routes)
- **Templates**: Use Jinja2 templates in `whyis/templates/` with the view infrastructure
- **Static files**: Place in `whyis/static/`
- **Configuration**: Use Flask configuration system
- **Views**: Prefer `current_app.render_view(resource)` over custom route handlers
- **Plugins**: Use Flask-PluginEngine for modular functionality

## Common Patterns

### Using the View Infrastructure

The preferred way to display resources is through the existing view infrastructure:

```python
from flask import current_app

# Get a resource by URI
resource = current_app.get_resource(entity_uri)

# Render the resource using the view system
# This automatically selects the appropriate template based on resource type
return current_app.render_view(resource)

# Render with a specific view
return current_app.render_view(resource, view='describe')
```

Resources are accessed via URI paths that get resolved by the view infrastructure. For example:
- `/about` - resolves to the "about" resource URI
- `/home` - resolves to the "home" resource URI  
- `/<path:name>` - resolves to any resource by its path

The view infrastructure automatically:
- Resolves the entity URI from the path
- Loads the resource and its properties from the knowledge graph
- Selects the appropriate template based on resource type

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tetherless-world/whyis](https://github.com/tetherless-world/whyis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
