---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Claude Code Custom Commands is a comprehensive collection of 45 custom slash commands for Claude Code that accelerate software development workflows through AI-powered automation. These commands provide intelligent automation for every stage of the software development lifecycle, from planning and architecture to deployment and monitoring.

## Core Philosophy

This project focuses on creating defensive security tools and development workflow automation. Each command leverages AI to analyze codebases and provide contextual assistance while maintaining security best practices.

### Key Principles:

1. **Security-First**: All commands focus on defensive security and safe development practices
2. **Workflow Automation**: Streamline repetitive development tasks with intelligent automation
3. **Comprehensive Coverage**: Support the entire software development lifecycle
4. **Quality Assurance**: Maintain high code quality through automated checks and validations
5. **Documentation-Driven**: Every command is thoroughly documented with usage examples

All commands are designed to enhance developer productivity while maintaining security and quality standards.

## Repository Structure

```
claude-code/
├── CLAUDE.md                           # This file - project guidance
├── README.md                           # Main project documentation
├── setup-devcontainer.sh               # Devcontainer setup script
├── claude-dev-toolkit/                 # NPM package (distributable toolkit)
│   ├── package.json                   # NPM package manifest
│   ├── bin/claude-commands            # CLI entry point
│   ├── lib/                           # JavaScript modules
│   ├── scripts/                       # Install/publish scripts
│   ├── commands/                      # Synced command copies for npm
│   ├── hooks/                         # Synced hook copies for npm
│   ├── subagents/                     # Synced subagent copies for npm
│   ├── templates/                     # Synced template copies for npm
│   └── tests/                         # NPM package tests
├── docs/                              # Documentation directory
│   ├── claude-custom-commands.md      # Command reference guide
│   ├── claude-code-hooks-system.md    # Hooks architecture documentation
│   ├── debug-context.md              # Debug context management
│   ├── devcontainer-guide.md          # Devcontainer guide
│   ├── manual-uninstall-install-guide.md # Installation/uninstallation guide
│   ├── npm-distribution-plan.md       # NPM distribution strategy
│   ├── npm-package-guide.md           # Published package information
│   ├── subagent-hook-integration.md   # Subagent integration docs
│   ├── npm-only/                      # NPM consolidation migration guides
│   ├── plans/                         # Implementation plans
│   └── publish/                       # Blog articles
├── hooks/                             # Hook implementations (10 shell + 14 Python)
│   ├── file-logger.sh                # File operation logging
│   ├── on-error-debug.sh             # Error debugging hook
│   ├── pre-commit-quality.sh         # Pre-commit quality checks
│   ├── pre-commit-test-runner.sh     # Auto-detect and run tests
│   ├── pre-write-security.sh         # Pre-write security validation
│   ├── prevent-credential-exposure.sh # Credential exposure prevention
│   ├── subagent-trigger.sh           # Subagent trigger hook (--simple for lightweight mode)
│   ├── tab-color.sh                  # Terminal tab colorization
│   ├── verify-before-edit.sh         # Warn about fabricated references
│   ├── claude-wrapper.sh             # Claude wrapper script
│   └── lib/                           # Hook support libraries (15 shell modules + 1 config)
│       ├── hook-helpers.sh           # Shared helpers for standalone hooks
│       ├── config-constants.sh        # Configuration constants
│       ├── file-utils.sh             # File utility functions
│       ├── error-handler.sh          # Error handling and logging
│       ├── argument-parser.sh        # CLI argument parsing
│       ├── context-manager.sh        # Context orchestrator (thin)
│       ├── context-gathering.sh      # Context data gathering
│       ├── context-file-ops.sh       # Context file I/O and validation
│       ├── execution-engine.sh       # Subagent execution engine
│       ├── execution-simulation.sh   # Execution simulation
│       ├── execution-results.sh      # Result processing
│       ├── subagent-discovery.sh     # Subagent discovery
│       ├── subagent-validator.sh     # Subagent validation
│       ├── field-validators.sh       # Field validation
│       ├── validation-reporter.sh    # Validation reporting
│       └── credential-patterns.conf  # Credential detection patterns
├── lib/                               # Shared utility libraries
│   └── logging.sh                    # Logging utilities
├── scripts/                           # Build and deployment scripts
│   ├── sync-to-npm.sh               # Sync source files to npm package
│   ├── deploy-subagents.sh          # Subagent deployment

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PaulDuvall/claude-code](https://github.com/PaulDuvall/claude-code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
