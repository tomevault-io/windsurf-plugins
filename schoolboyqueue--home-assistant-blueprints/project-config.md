---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This repository contains production-ready Home Assistant Blueprints for home automation. Blueprints are YAML files with Jinja2 templating that define reusable automation templates for Home Assistant.

See [CONTRIBUTING.md](CONTRIBUTING.md) for contribution guidelines.

## Commands

### Make Targets

| Target                             | Description                                                |
| ---------------------------------- | ---------------------------------------------------------- |
| `make setup`                       | Setup development environment (pre-commit, Go tools, docs) |
| `make validate`                    | Validate all blueprints                                    |
| `make validate-single FILE=<path>` | Validate a single blueprint                                |
| `make build`                       | Build all Go tools                                         |
| `make go-init`                     | Download Go dependencies                                   |
| `make go-tools`                    | Install Go dev tools (golangci-lint, gofumpt, goimports)   |
| `make go-test`                     | Run Go tests                                               |
| `make go-lint`                     | Run Go linters (with auto-fix)                             |
| `make go-format`                   | Format Go code                                             |
| `make go-vet`                      | Run go vet                                                 |
| `make go-check`                    | Run all Go checks (format, lint, vet, test)                |
| `make go-audit`                    | Run security audit with govulncheck                        |
| `make go-clean`                    | Clean Go build artifacts                                   |
| `make docs-check`                  | Check docs with Biome                                      |
| `make docs-fix`                    | Fix docs issues with Biome                                 |
| `make markdown-check`              | Check markdown files with markdownlint                     |
| `make markdown-lint`               | Lint and fix markdown files with markdownlint              |
| `make markdown-format`             | Format markdown tables with prettier                       |
| `make markdown-fix`                | Lint and format all markdown files                         |
| `make check`                       | Run all checks (Go + blueprints + docs + markdown)         |
| `make check-all`                   | Run all checks including security audit                    |
| `make clean`                       | Clean build artifacts                                      |
| `make help`                        | Show all available targets                                 |

### Validate Blueprints

```bash
# Validate a single blueprint
./scripts/validate-blueprint-go/build/validate-blueprint <path/to/blueprint.yaml>

# Validate all blueprints in the repository
./scripts/validate-blueprint-go/build/validate-blueprint --all

# Or use make targets
make validate                           # Validate all blueprints
make validate-single FILE=<path>        # Validate a single file
```

The validator checks:

- YAML syntax and blueprint schema
- Input/selector definitions and !input reference validation
- Template syntax (balanced delimiters, no !input inside {{ }})
- Service call structure
- Version sync (blueprint name vs blueprint_version variable)
- Trigger validation (no templates in `for:` duration)
- Condition structure validation
- Mode validation (single, restart, queued, parallel)
- Delay and wait_template/wait_for_trigger validation
- Empty sequence detection
- README.md and CHANGELOG.md existence

## Architecture

### Blueprint Structure

Each blueprint lives in `blueprints/<blueprint-name>/` and contains:

- `*.yaml` - The blueprint file (named `*_pro.yaml` or `*_pro_blueprint.yaml`)
- `README.md` - Documentation
- `CHANGELOG.md` - Version history

### Blueprint YAML Structure

```yaml
blueprint:
  name: "Blueprint Name vX.Y.Z"
  description: >-
    Multi-line description
  domain: automation
  author: "Author Name"
  source_url: https://github.com/...
  input:
    group_name:
      name: Group Label
      icon: mdi:icon-name
      input:
        input_name:
          name: Input Label
          description: Description
          default: value
          selector:
            selector_type: options...

variables:
  blueprint_version: "X.Y.Z"
  # Variables defined here, referenced in templates

trigger:
  - platform: state
    entity_id: !input input_name
    # ...

action:
  - if:
      - condition: template
        value_template: "{{ expression }}"
    then:
      - service: domain.service
        target:
          entity_id: !input target_input
```

### Key Patterns

1. **!input tags**: Use `!input input_name` to reference blueprint inputs. Cannot be used inside `{{ }}` templates - bind to a variable first

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [schoolboyqueue/home-assistant-blueprints](https://github.com/schoolboyqueue/home-assistant-blueprints) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-30 -->
