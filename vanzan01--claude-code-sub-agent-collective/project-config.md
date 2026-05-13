---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the **Claude Code Sub-Agent Collective** - an NPX-distributed framework that installs specialized AI agents, hooks, and behavioral systems for TDD-focused development workflows. The system enforces test-driven development through automated handoff validation and provides intelligent task routing through a hub-and-spoke architecture.

## CRITICAL REPOSITORY INFORMATION

**Git Remote URL:** https://github.com/vanzan01/claude-code-sub-agent-collective.git
**NEVER CHANGE THIS URL** - Always use this exact repository URL for all git operations.

## Architecture

### Core System
- **Hub-and-spoke architecture** with `@routing-agent` as central coordinator
- **Behavioral Operating System** defined in `CLAUDE.md` with prime directives
- **Test-Driven Handoffs** with contract validation between agents
- **Just-in-time context loading** to minimize memory usage

### Key Components
- **NPX Package**: `claude-code-collective` - Installable via `npx claude-code-collective init`
- **Agent System**: 30+ specialized agents in `templates/agents/`
- **Hook System**: TDD enforcement hooks in `templates/hooks/`
- **Command System**: Natural language + structured commands in `lib/command-*.js`
- **Metrics Framework**: Research hypotheses tracking in `lib/metrics/`
- **Template System**: Installation templates in `templates/`

## Essential Commands

### Development
```bash
# Run tests (primary test suite)
npm test                    # Vitest tests
npm run test:jest          # Jest tests (comprehensive)
npm run test:coverage      # Coverage reports

# Run specific test suites  
npm run test:contracts     # Contract validation tests
npm run test:handoffs      # Agent handoff tests
npm run test:agents        # Agent system tests

# Package management
npm run install-collective # Install to current directory
npm run validate          # Validate installation
npm run metrics:report    # View metrics data
```

### Local Testing Workflow

For testing changes before publishing (see ai-docs/Simple-Local-Testing-Workflow.md):

```bash
# Automated testing (does everything automatically)
./scripts/test-local.sh
# This script automatically:
# - Creates package (.tgz file)  
# - Creates test directory ../npm-tests/ccc-testing-vN (auto-numbered)
# - Installs the package in test directory
# - Runs basic validation tests
# - Leaves you in the test directory ready for more testing

# Additional manual testing (you're already in test directory after script)
npx claude-code-collective init            # Interactive mode
npx claude-code-collective init --minimal  # Minimal installation  
npx claude-code-collective --help          # Help information

# Return to main directory and cleanup when done
cd ../taskmaster-agent-claude-code
./scripts/cleanup-tests.sh # Removes test directories and tarballs
```

#### Testing Scripts Available
- `scripts/test-local.sh` - Automated package testing in dedicated `../npm-tests/` directory
- `scripts/cleanup-tests.sh` - Clean up test artifacts and directories (removes npm-tests when empty)

#### NPM Testing Directory Naming Standards

**MANDATORY NAMING CONVENTION**: All npm testing directories MUST follow the established pattern:

- **Manual testing**: `ccc-manual-v[N]` (e.g., `ccc-manual-v1`, `ccc-manual-v2`)
- **Automated testing**: `ccc-automated-v[N]` (e.g., `ccc-automated-v1`, `ccc-automated-v2`) 
- **Feature-specific testing**: `ccc-[feature]-v[N]` (e.g., `ccc-backup-test-v1`, `ccc-hooks-test-v1`)

**DO NOT** use arbitrary names like `test-backup-validation` or any other format. Always use the `ccc-*` prefix followed by descriptive name and version number.

### NPX Package Testing
```bash
# Test the NPX package locally (quick testing)
npx . init                 # Test installation from current directory
npx . status              # Test status command
npx . validate            # Test validation
```

## Key Development Files

### Core Implementation
- `lib/index.js` - Main entry point and ClaudeCodeCollective class
- `lib/installer.js` - NPX installation logic
- `lib/command-system.js` - Natural language command processing
- `lib/AgentRegistry.js` - Agent management and lifecycle
- `bin/claude-code-collective.js` - CLI interface

### Testing Infrastructure
- `jest.config.js` - Jest configuration for comprehensive testing
- `vitest.config.js` - Vitest configuration for fast iteration
- `tests/setup.js` - Test environment setup
- `tests/contracts/` - Contract validation tests
- `tests/handoffs/` - Agent handoff tests

### Templates and Distribution
- `templates/` - All installation templates (agents, hooks, configs)
- `templates/CLAUDE.md` - Behavioral system template
- `templates/settings.json` - Claude Code configuration template
- `lib/file-mapping.js` - Template to destination mapping

## Development Workflow

### Branch-Based Testing Workflow

**Standard process for testing changes before merging:**

1. **Create Feature Branch**
   ```bash
   git checkout -b feature/your-feature-name
   # Make changes...
   git add . && git commit -m "feat: your changes"
   ```

2. **Test Locally** 
   ```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vanzan01/claude-code-sub-agent-collective](https://github.com/vanzan01/claude-code-sub-agent-collective) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
