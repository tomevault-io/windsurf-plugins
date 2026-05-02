---
trigger: always_on
description: Composite actions for comprehensive security scanning, designed for GitHub Enterprise Server (GHES) with github.com access.
---

# Argus

Composite actions for comprehensive security scanning, designed for GitHub Enterprise Server (GHES) with github.com access.

---

## Project Vision & Goals

**Primary Vision**: Make it as easy as possible for users to employ a hardening pipeline on their projects to gain insights into their security footprint and current vulnerabilities.

### Core Principles

1. **Documentation serves both humans and AI**: Concise, clear, on point. Structured context in `.ai/` for machine-readability.
2. **Code must be simple and maintainable**: Minimize complexity, maximize clarity. Easy for anyone to understand and extend.
3. **Dependabot is foundational**: Automated dependency updates are critical to the pipeline's value.
4. **Trust is everything**: The pipeline must earn trust in PRs through extremely robust testing. Only then can it auto-merge and auto-release.
5. **Automation end-state**:
   - Dependabot dependency updates arrive in PRs
   - Pipeline runs automatically
   - Green test results = trusted
   - Auto-merge enabled
   - Auto-release to users

---

## Testing Philosophy

**Just completed migration: ALL action scripts and tests are now Python.**

### Standards

- **Single Language**: Python for all action scripts and tests (not Bash, not Node.js for actions)
- **Single Test Framework**: pytest (not jest, mocha, or other)
- **Single Coverage Tool**: pytest-cov (with `--cov-fail-under=80` in pytest.ini)
- **Minimum Coverage**: 80% enforced at all times

### Test Structure

```
.github/actions/scanner-{name}/
├── scripts/
│   ├── parse-results.py       # Scanner output → JSON
│   └── generate-summary.py    # JSON → Markdown
└── tests/
    ├── test_parse_results.py
    ├── test_generate_summary.py
    └── conftest.py (optional)

tests/
├── fixtures/
│   └── scanner-outputs/       # Pre-captured real scanner results
└── (integration tests)
```

### Test Execution

```bash
# Full run with coverage (enforced: ≥80%)
pytest

# Fast validation (no coverage)
pytest --no-cov -q

# Specific action
pytest .github/actions/scanner-clamav/tests/
```

### Reference Implementation

**`scanner-clamav`** is the reference pattern for:
- Python action script structure
- Test organization and fixtures
- Coverage targets (80%+)
- How to test scanner parsing and summary generation

All new scanner actions should follow this exact pattern.

---

## Project Conventions

### Versioning & Release

- **Single Version Source**: `version.yaml` (prevents drift)
- **Release Command**: `npm run release` (manages all version updates and tags)
- **Version Tags**: Release workflow auto-tags and publishes

### Commit Messages

Follow **Conventional Commits**:
```
feat(scanner-name): add support for X
fix(parser): handle empty results
docs: update scanner reference
test(bandit): add edge case coverage
refactor: simplify parse logic
```

### Release Process

Users depend on you auto-releasing after dependency updates pass. The testing pipeline must be bulletproof for this trust.

---

## AI Context Ecosystem

This project uses **AI Context as Code (AICaC)** - structured, machine-readable context in `.ai/`:

| File | Purpose |
|------|---------|
| `.ai/context.yaml` | Project metadata and entry points |
| `.ai/architecture.yaml` | Component relationships and dependencies |
| `.ai/workflows.yaml` | Common tasks with exact commands |
| `.ai/decisions.yaml` | Architectural Decision Records (ADRs) |
| `.ai/errors.yaml` | Error patterns and solutions |
**Reading order**: `.ai/context.yaml` → relevant module files → source code

### CRITICAL: Maintain .ai/ Files

**After making changes to this project, you MUST update the relevant `.ai/` files.**

| When you change... | Update... |
|--------------------|-----------|
| Components/structure | `.ai/architecture.yaml` |
| Commands/tasks | `.ai/workflows.yaml` |
| Make design decisions | `.ai/decisions.yaml` |
| Fix common errors | `.ai/errors.yaml` |
| Project metadata | `.ai/context.yaml` |
| Scanners or actions | `.ai/architecture.yaml` (scanners list + components) |
| Version number | `.ai/context.yaml` (version field) |

**Before completing any task**, verify:
```
[ ] Relevant .ai/ files updated (or confirmed not needed)
```

---

## AI Assistant Configuration

### Global Standards (Claude Code)

Claude Code users: Global rules, skills, and agents from `~/.claude/` are automatically applied. These include:

- **Rules**: coding-style, git-workflow, testing, security, performance, refactor-clean
- **Skills**: security-skills, documentation-skills, data-analysis-skills
- **Agents**: planner, security-reviewer, technical-docs-writer

Source: [huntridge-labs/cheat-codes](https://github.com/huntridge-labs/cheat-codes)

### Project Overrides

To override global settings for this project, create `.claude/settings.json`:

```json
{
  "rules": {
    "disabled": ["performance"],
    "project_specific": true
  }
}
```

Or add project-specific rules in `.claude/rules/`.

### Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `CLAUDE_SKIP_GLOBAL_RULES` | Skip loading ~/.claude/rules/ | `false` |
| `CLAUDE_VERBOSE` | Show which rules are being applied | `false` |

### GitHub Copilot Users


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [huntridge-labs/argus](https://github.com/huntridge-labs/argus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
