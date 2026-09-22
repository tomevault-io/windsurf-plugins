---
trigger: always_on
description: This is the Yoto Smart Stream project - a service to stream audio to Yoto devices, monitor events via MQTT, and manage interactive audio experiences. The project includes Python/FastAPI backend, web UI components, and Railway deployment infrastructure.
---

# GitHub Copilot Workspace Instructions

## Project Overview

This is the Yoto Smart Stream project - a service to stream audio to Yoto devices, monitor events via MQTT, and manage interactive audio experiences. The project includes Python/FastAPI backend, web UI components, and Railway deployment infrastructure.

## Guardrails
- **NEVER** write outside of the workspace. use tmp/ in the workspace for temporary files.

## Code Style and Conventions

- **Language**: Python 3.9+
- **Framework**: FastAPI with async/await patterns
- **Testing**: pytest with fixtures, mocks, and comprehensive coverage (target >70%)
- **Linting**: ruff for code quality, black for formatting
- **Type hints**: Use Python typing throughout the codebase

## Design Principles
### Modal Dialogs
- Modal dialogs should close with the Escape key.

## Locally-Maintained Skills

This workspace contains locally-maintained custom skills in `.github/skills/`. These skills are **specialized agents with domain expertise** that should be used whenever relevant work is being done.

### Available Skills

1. **railway-service-management** - Multi-environment Railway deployment management
   - **Use for**: Railway deployments, environment management, CLI operations, configuration
   - **Key tools**: `get_deployment_endpoint.py` script for retrieving deployment URLs
   - **Reference**: `.github/skills/railway-service-management/SKILL.md`

2. **yoto-smart-stream** (formerly yoto-api-development) - Yoto Play API integration, audio streaming, and MQTT handling
   - **Use for**: Yoto API integration, MQTT events, audio streaming, service operations
   - **Reference**: `.github/skills/yoto-smart-stream/SKILL.md`

3. **yoto-smart-stream-testing** - Comprehensive testing for Yoto Smart Stream
   - **Use for**: Writing tests, debugging test failures, authentication testing, Playwright UI automation
   - **Reference**: `.github/skills/yoto-smart-stream-testing/SKILL.md`

### When to Use Skills

**ALWAYS delegate to the appropriate skill when working on:**

- 🚂 **Railway tasks**: Deployments, environment setup, getting endpoint URLs, configuration
  → Use `railway-service-management` skill

- 🎵 **Yoto API tasks**: API integration, MQTT handling, audio streaming, device management
  → Use `yoto-smart-stream` skill

- 🧪 **Testing tasks**: Writing tests, fixing test failures, authentication flows, UI testing
  → Use `yoto-smart-stream-testing` skill

**Examples:**
- Deploying to Railway environment → Invoke `railway-service-management` skill
- Getting a Railway deployment URL → Use `.github/skills/railway-service-management/scripts/get_deployment_endpoint.py`
- Implementing Yoto API endpoint → Invoke `yoto-smart-stream` skill
- Writing integration tests → Invoke `yoto-smart-stream-testing` skill
- Fixing OAuth login flow → Invoke `yoto-smart-stream-testing` skill

### Skill Maintenance Directive

**IMPORTANT**: When you discover, verify, or implement new information related to these skills during development or issue resolution, you MUST update the relevant skill documentation to keep it current and accurate.

**When to Update Skills:**

- When you verify new API endpoints, parameters, or behaviors
- When you discover Railway deployment patterns or configuration options
- When you implement new Yoto API features or MQTT event handling
- When you find corrections to existing documentation
- When you establish new best practices or patterns
- When you resolve issues that reveal gaps in the skill documentation

**How to Update Skills:**

1. **Identify the relevant skill** - Determine which skill (`railway-service-management` or `yoto-smart-stream`) the new information relates to
2. **Locate the appropriate file** - Skills have a main `SKILL.md` and reference docs in `reference/` subdirectories
3. **Update the documentation** - Add or correct information in the relevant file(s)
4. **Be specific** - Include examples, code snippets, or command syntax where applicable
5. **Maintain consistency** - Follow the existing structure and formatting of the skill files
6. **Update related sections** - If the change affects multiple areas, update all relevant sections

**Skill File Structure:**

```
.github/skills/
├── railway-service-management/
│   ├── SKILL.md                    # Main overview and quick start
│   └── reference/                  # Detailed reference docs
│       ├── platform_fundamentals.md
│       ├── deployment_workflows.md
│       ├── configuration_management.md
│       └── ...
├── yoto-smart-stream/
│   ├── SKILL.md                    # Main overview and quick start
│   └── reference/                  # Detailed reference docs
│       ├── yoto_api_reference.md
│       ├── yoto_mqtt_reference.md
│       ├── architecture.md
│       ├── service_operations.md
│       └── ...
└── yoto-smart-stream-testing/
    ├── SKILL.md                    # Main overview and quick start
    └── reference/                  # Detailed reference docs
        ├── testing_guide.md
        └── login_workflows.md
```

**Examples of Updates to Make:**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [earchibald/yoto-smart-stream](https://github.com/earchibald/yoto-smart-stream) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
