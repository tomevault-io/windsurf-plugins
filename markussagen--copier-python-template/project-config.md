---
trigger: always_on
description: This document provides instructions for working with the copier-python-template template itself.
---

# Claude Code Instructions for copier-python-template Template

This document provides instructions for working with the copier-python-template template itself.

## Project Overview

**copier-python-template** is a modern Python project template that generates projects following the Arkalos directory structure. It combines best practices from copier-uv with a comprehensive, production-ready setup.

## Template Structure

```
copier-python-template/
├── copier.yml              # Template configuration and questions
├── extensions.py           # Jinja2 custom filters and extensions
├── project/                # Template files (all .jinja suffixed)
│   ├── app/               # Arkalos app structure
│   ├── config/            # Configuration templates
│   ├── tests/             # Test templates
│   ├── docs/              # Documentation templates
│   ├── .github/           # GitHub Actions workflows
│   ├── pyproject.toml.jinja
│   ├── Makefile.jinja
│   ├── Dockerfile.jinja
│   ├── compose.yaml.jinja
│   ├── .mise.toml.jinja
│   ├── .pre-commit-config.yaml.jinja
│   └── CLAUDE.md.jinja
├── CLAUDE.md              # This file
└── readme.md              # Template documentation
```

## Modifying the Template

### Adding New Questions

Edit `copier.yml`:

```yaml
new_option:
  type: bool
  help: Enable new feature?
  default: false
```

### Adding Conditional Features

Use Jinja2 conditionals in template files:

```jinja2
{%- if new_option %}
# Feature-specific code
{%- endif %}
```

### Adding Dependencies

In `pyproject.toml.jinja`:

```jinja
{%- if new_option %}
[project.optional-dependencies]
new_feature = [
    "package>=1.0.0",
]
{%- endif %}
```

### Adding Directories

Use conditional directory names:

```
project/app/{% if new_option %}new_feature{% endif %}/
```

## Testing the Template

### Generate a Test Project

```bash
# From parent directory
copier copy --trust copier-python-template test-project

# Answer prompts, then test the generated project
cd test-project
make setup
make lint
make test
```

### Test with All Options

```bash
# Create a test script to try all combinations
copier copy --trust \
  -d project_name="Test Project" \
  -d include_http=true \
  -d include_ai=true \
  -d include_docker=true \
  copier-python-template test-all-features
```

## Template Features

### Core Features (Always Included)

- **Build system**: Hatchling + hatch-vcs for git-based versioning
- **Package manager**: uv for fast, reliable dependency management
- **Tool manager**: mise for Python version management
- **Code quality**: Ruff (linting + formatting)
- **Type checking**: ty (fast) + mypy (comprehensive)
- **Testing**: pytest with coverage
- **Pre-commit**: prek (Rust-based, faster than pre-commit)
- **Documentation**: MkDocs Material
- **CI/CD**: GitHub Actions
- **Structure**: Arkalos project layout

### Optional Features (User-Selected)

- **HTTP/Web** (`include_http`):
  - FastAPI setup with routes/controllers/middleware
  - Health check endpoint
  - Docker support
  - API testing fixtures

- **AI/ML** (`include_ai`):
  - AI directories (agents, actions, evals, trainers)
  - ML dependencies (numpy, scikit-learn)
  - AI dependencies (openai, anthropic)

- **Data Processing** (`include_data_processing`):
  - Data directories (extractors, transformers, analyzers, visualizers)
  - Data dependencies (pandas, pyarrow, polars)

- **Hardware** (`include_hardware`):
  - Hardware directories (sensors, actuators, controllers, communicators)

- **Docker** (`include_docker`):
  - Multi-stage Dockerfile
  - docker-compose.yaml
  - .dockerignore

- **DevContainer** (`include_devcontainer`):
  - VS Code devcontainer configuration
  - Pre-configured extensions
  - Automatic environment setup

- **CLI** (`python_package_command_line_name`):
  - Typer-based CLI setup
  - Entry point configuration

## Jinja2 Extensions

Located in `extensions.py`:

### GitExtension

Filters to auto-detect git configuration:

```jinja2
{{ 'Default Name' | git_user_name }}
{{ 'default@email.com' | git_user_email }}
```

### SlugifyExtension

Convert strings to URL-safe slugs:

```jinja2
{{ project_name | slugify }}        # my-project
{{ project_name | slugify('_') }}   # my_project
```

### CurrentYearExtension

Global variable for current year:

```jinja2
{{ current_year }}  # 2026
```

## Best Practices for Template Development

### 1. Use Conditional Directories

Don't create directories for features the user didn't select:

```
app/{% if include_http %}http{% endif %}/
```

### 2. Use Optional Dependencies

Keep core dependencies minimal, make feature-specific deps optional:

```toml
[project.optional-dependencies]
web = ["fastapi[standard]>=0.115.0"]
```

### 3. Provide Good Defaults

Auto-fill derived values:

```yaml
python_package_import_name:
  default: "{{ project_name | slugify('_') }}"
```

### 4. Test All Combinations

Test with different feature combinations to ensure they work together.

### 5. Document Everything

- Update CLAUDE.md for generated projects
- Update README.md for the template itself
- Add comments in complex Jinja2 templates

## Updating from copier-uv

This template is based on copier-uv but with significant modifications:

### Key Differences


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MarkusSagen) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
