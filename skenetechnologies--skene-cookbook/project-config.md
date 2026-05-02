---
trigger: always_on
description: **Repository:** skene-cookbook - 764 AI skill library with 36 skill chain recipes
---

# Cursor Rules for skene-cookbook

## Project Context

**Repository:** skene-cookbook - 764 AI skill library with 36 skill chain recipes
**Package:** @skene/skills-directory v0.2.0
**License:** MIT
**Purpose:** Pre-built AI skill chains for PLG, sales, customer success, security, and more

## Architecture Patterns

### Skill Structure (Non-Negotiable)

Every skill MUST follow this exact structure:

```
skills-library/{domain}/{skill-name}/
├── skill.json          # Metadata (name, description, category, risk_level)
├── instructions.md     # AI agent execution instructions
└── tests/              # Optional: skill-specific tests
```

**Do NOT:**
- Create skills outside `skills-library/executable/` or `skills-library/reference/`
- Modify existing skill structure without updating registry
- Add skills without `skill.json` and `instructions.md`

### Registry Auto-Generation

The `registry/` directory is AUTO-GENERATED from `skills-library/`:
- Never manually edit files in `registry/`
- Always run `npm run verify:metrics` after skill changes
- Registry regeneration syncs badges and counts across README.md, METRICS.md, docs/directory.md

### Risk Level Classification

Skills are classified by risk level (defined in `skill.json`):
- `Low` - Read-only operations, no external dependencies
- `Medium` - Write operations, requires configuration
- `High` - External API calls, requires credentials
- `Critical` - System-level operations, requires manual review

**Rule:** When creating a skill that makes external API calls, uses credentials, or modifies data, always set `risk_level: "High"` or `"Critical"`.

## Naming Conventions

### Skill IDs
- Format: `{domain}_{action}_{target}` (e.g., `sales_analyze_pipeline`)
- Use snake_case, all lowercase
- Be descriptive but concise (3-5 words max)

### File Naming
- Skill directories: kebab-case (e.g., `analyze-customer-health/`)
- Python files: snake_case (e.g., `analyze_skills.py`)
- JavaScript files: kebab-case (e.g., `skills-directory.js`)
- Documentation: SCREAMING_SNAKE_CASE for top-level (e.g., `AGENTS.md`), kebab-case for nested

### Domain Categories

Valid domains (don't invent new ones without discussion):
- `ecosystem` - Partner, integration management
- `marketing` - Campaigns, content, SEO
- `sales` - Pipeline, deals, CRM
- `customer_success` - Health, churn, onboarding
- `product_ops` - Roadmap, releases
- `security` - Security, compliance
- `finops` - Billing, revenue
- `data` - Analytics, reporting
- `engineering` - DevOps, CI/CD
- `hr` - Recruiting, onboarding

## Testing Patterns

### Test Organization

```
tests/
├── unit/           # Fast, isolated tests (< 1s each)
├── integration/    # Multi-component tests (< 5s each)
└── e2e/            # Full user workflows (< 30s each)
```

### Required Test Coverage

- Minimum: 60% overall coverage
- Target: 80%+ coverage
- Critical paths (eval_harness, tracer): 90%+ coverage

### Testing Commands

```bash
# Run full suite
pytest tests/ -v

# Run fast tests only
pytest tests/unit -v -m "not slow"

# Run specific domain tests
pytest tests/unit/test_dedupe_skills.py -v
```

**Rule:** All PRs must pass `pytest tests/ -v` before merging.

## Code Quality Standards

### Pre-Commit Hooks (Enforced)

The following hooks run automatically on commit:
- `detect-secrets` - Blocks commits with credentials
- `prettier` - Formats JS/JSON/YAML/Markdown
- `black` - Formats Python (if installed)
- `flake8` - Lints Python (if installed)
- `isort` - Sorts Python imports (if installed)

**Rule:** Never use `--no-verify` to skip hooks. Fix the issues instead.

### Linting Commands

```bash
# JavaScript
npm run lint        # ESLint + Prettier
npm run format      # Auto-fix formatting

# Python (if installed in venv)
black .
flake8 .
isort .
```

## Workflow Blueprints

### Blueprint Schema

Blueprints in `registry/blueprints/` follow `schemas/workflow_blueprint.json`:

```yaml
id: workflow_{name}
version: 1.0.0
name: 'Human Readable Name'
chain_sequence:
  - step_id: 'step_1'
    skill_id: 'domain_action_target'
    action: 'action_name'
    input_mapping:
      static_values: {}
    error_handling:
      on_failure: 'stop'
      max_retries: 2
```

**Rule:** All workflow blueprints must validate against schema before committing.

## Playbook-Ready Features (Optional but Encouraged)

When creating blueprints, consider adding:

```yaml
icp:
  company_size: '50-500'
  motion: 'product-led-growth'
  priorities: ['reduce_churn', 'increase_nrr']

integration_reference:
  - type: 'crm'
    provider: 'salesforce'
    schema_ref: 'registry/integration_schemas/salesforce_fields.yaml'

opinionated_prompts:
  - step_id: 'step_1'
    system_context: 'You are analyzing a PLG motion with 30-day trials...'
    input_guidance: 'Focus on trial-to-paid conversion metrics...'
```

See `registry/integration_schemas/README.md` for schema format.

## Security Rules (Blocking)

### Never Commit

These patterns are BLOCKED by pre-commit hooks:
- `.env` files (use `.env.example` for templates)
- Files in `.ssh/`, `.aws/`, `secrets/`
- Private keys (detected by `detect-private-key` hook)
- API keys, tokens, passwords (detected by `detect-secrets`)

### Handling Credentials

```bash
# Good: Reference environment variables

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SkeneTechnologies/skene-cookbook](https://github.com/SkeneTechnologies/skene-cookbook) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
