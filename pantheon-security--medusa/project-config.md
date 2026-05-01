---
trigger: always_on
description: Always use `dangerouslyDisableSandbox: true` on Bash tool calls that run `medusa scan` or invoke external linters. The default sandbox strips `~/.npm-global/bin`, `~/.local/bin`, and `/snap/bin` from PATH, causing installed tools (eslint, hadolint, rubocop, trivy, etc.) to appear missing and triggering the "Continue scan without these tools?" prompt which blocks non-interactive scans.
---

# medusa - MEDUSA Security Scanning

## CRITICAL: Bash Sandbox

Always use `dangerouslyDisableSandbox: true` on Bash tool calls that run `medusa scan` or invoke external linters. The default sandbox strips `~/.npm-global/bin`, `~/.local/bin`, and `/snap/bin` from PATH, causing installed tools (eslint, hadolint, rubocop, trivy, etc.) to appear missing and triggering the "Continue scan without these tools?" prompt which blocks non-interactive scans.

## Project Overview

This project uses **MEDUSA v2026.5.0** - AI Security Scanner with 9,600+ detection patterns for AI/ML, agents, and LLM applications. Works out of the box with no external tool installation required.

## MEDUSA Configuration

**Location**: `.medusa.yml`

### Quick Commands

```bash
# Run security scan (works immediately - no setup needed)
medusa scan .

# Quick scan (cached results)
medusa scan . --quick

# Exclude directories from scan
medusa scan . -e archive/ -e vendor/

# Check tool status
medusa install --check

# Install AI tools (modelscan for ML model scanning)
medusa install --ai-tools

# License management
medusa license info        # View license status
medusa license activate    # Activate license key
medusa license trial       # Start 14-day trial
medusa license deactivate  # Remove license
```

## Available Slash Commands

- `/medusa-scan` - Run security scan on project
- `/medusa-install` - Install missing security tools

## Integration Features

### Claude Code Integration

- **Auto-scan on save**: Automatically scans files when you save them
- **Inline annotations**: Security issues appear directly in your IDE
- **Smart detection**: Only scans relevant file types
- **Parallel processing**: Fast scanning with multi-core support

### AI-First Security

MEDUSA scans with 9,600+ built-in patterns for:
- AI/ML applications, LLM agents, MCP servers
- Prompt injection, RAG poisoning, agent security
- Traditional vulnerabilities (SQL injection, XSS, secrets)
- Configuration files (YAML, JSON, Terraform, Docker)

**Optional**: External linters (bandit, eslint, etc.) are auto-detected if installed.

## Security Scanning

### Scan Reports

Reports are generated in `.medusa/reports/`:
- HTML dashboard (visual report)
- JSON data (for CI/CD integration)
- SARIF output (GitHub integration)
- CLI output (terminal summary)

### Output Formats

```bash
# Default JSON output
medusa scan . --output json

# SARIF format (GitHub Code Scanning)
medusa scan . --output sarif

# HTML dashboard
medusa scan . --output html
```

### Severity Levels

- **CRITICAL**: Immediate security threats
- **HIGH**: Significant vulnerabilities
- **MEDIUM**: Moderate issues
- **LOW**: Minor concerns
- **INFO**: Best practice suggestions

### Fail Thresholds

Configure scan to fail CI/CD on certain severity:

```bash
medusa scan . --fail-on high
```

## Configuration

Edit `.medusa.yml` to customize:

```yaml
version: 2026.3.0
scanners:
  enabled: []     # Empty = all enabled
  disabled: []    # List scanners to disable
fail_on: high     # critical | high | medium | low
exclude:
  paths:
    - node_modules/
    - .venv/
    - dist/
workers: null     # null = auto-detect CPU cores
cache_enabled: true
output_format: sarif  # json | sarif | html
```

## CI/CD Integration

### GitHub Actions

```yaml
name: MEDUSA Security Scan
on: [push, pull_request]

jobs:
  security-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Run MEDUSA Scan
        uses: pantheon-security/medusa-action@v2026
        with:
          fail-on: high
          output-format: sarif

      - name: Upload SARIF results
        uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: .medusa/reports/results.sarif
```

### GitLab CI

```yaml
security_scan:
  script:
    - pip install medusa-security
    - medusa scan . --fail-on high --output sarif
  artifacts:
    reports:
      sast: .medusa/reports/results.sarif
```

## Licensing and Pricing

### Tier Comparison

| Feature | FREE | Professional | Enterprise |
|---------|------|--------------|------------|
| AI Security Patterns | 9,600+ | 9,600+ | 9,600+ |
| Runtime Filters | - | 1,100+ | 1,100+ |
| SARIF Output | Yes | Yes | Yes |
| CLI | Yes | Yes | Yes |
| GitHub Action | Yes | Yes | Yes |
| REST API | - | Yes | Yes |
| Webhooks | - | Yes | Yes |
| Custom Rules | - | - | Yes |
| SSO/SAML | - | - | Yes |
| Audit Logs | - | - | Yes |
| **Price** | Free | $99/dev/mo | $499/50 devs/mo |

### License Commands

```bash
# Check current license status
medusa license info

# Activate a license key
medusa license activate YOUR-LICENSE-KEY

# Start a 14-day Professional trial
medusa license trial

# Deactivate license (for transferring)
medusa license deactivate
```

### Runtime Filters (Professional/Enterprise)

Runtime filters provide 1,100+ additional rules for detecting:
- AI/ML model attacks and vulnerabilities
- Prompt injection patterns

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Pantheon-Security/medusa](https://github.com/Pantheon-Security/medusa) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
