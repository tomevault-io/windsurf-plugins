---
trigger: always_on
description: This repository contains the Train Transport Interface, a Ruby library that provides a unified interface to talk to local or remote operating systems and APIs. Train is a core component of the Chef InSpec ecosystem.
---

# GitHub Copilot Instructions for Train Repository

## Repository Overview

This repository contains the Train Transport Interface, a Ruby library that provides a unified interface to talk to local or remote operating systems and APIs. Train is a core component of the Chef InSpec ecosystem.

### Folder Structure
```
train/
├── .github/                              # GitHub workflows and configurations
│   ├── CODEOWNERS                        # Code ownership definitions
│   ├── ISSUE_TEMPLATE/                   # Issue templates
│   ├── dependabot.yml                    # Dependabot configuration
│   ├── lock.yml                          # Lock configuration
│   └── workflows/                        # CI/CD workflows
├── .expeditor/                           # Chef Expeditor CI configuration
├── contrib/                              # Contribution utilities
├── docs/                                 # Documentation
├── examples/                             # Example code and plugins
├── lib/                                  # Core Train library code
│   ├── train/                            # Main Train modules
│   │   ├── extras/                       # Additional utilities
│   │   ├── file/                         # File handling modules
│   │   ├── platforms/                    # Platform detection
│   │   ├── plugins/                      # Plugin system
│   │   └── transports/                   # Transport implementations
│   │       ├── clients/                  # Transport clients
│   │       └── helpers/                  # Transport helpers
│   └── train.rb                          # Main entry point
├── test/                                 # Test suites
│   ├── fixtures/                         # Test fixtures and plugins
│   ├── integration/                      # Integration tests
│   ├── unit/                             # Unit tests (Minitest)
│   └── windows/                          # Windows-specific tests
├── Gemfile                               # Ruby dependencies
├── Rakefile                              # Rake tasks
├── train.gemspec                         # Gem specification
├── train-core.gemspec                    # Core gem specification
└── README.md                             # Project documentation
```

### Key Technologies
- **Ruby**: Primary language (Ruby 2.7+)
- **Minitest**: Primary testing framework
- **SimpleCov**: Code coverage tool
- **Bundler**: Ruby dependency management
- **Rake**: Ruby build tool
- **Mocha**: Mocking framework
- **ChefStyle**: Ruby code style enforcement and linting

### Supported Transports
- Local execution
- SSH
- WinRM
- Docker and Podman
- Mock (for testing)
- AWS API
- Azure API
- VMware via PowerCLI
- Habitat

## Critical Instructions

### 🚨 File Modification Restrictions
- **DO NOT modify any `*.codegen.go` files** if present in the repository
- These are auto-generated files and should never be manually edited
- Always check for presence of codegen files before making changes

### JIRA Integration & Task Implementation Workflow

When a JIRA ID is provided, follow this complete workflow:

#### 1. JIRA Issue Analysis
- Use the `atlassian-mcp-server` MCP server to fetch JIRA issue details
- Read and understand the story requirements thoroughly
- Identify all acceptance criteria and technical requirements
- Note any dependencies or constraints mentioned
- Consider transport-specific requirements and compatibility

#### 2. Implementation Planning
- Break down the task into smaller, manageable components
- Identify which files need to be created, modified, or tested
- Plan the implementation approach based on Train's architecture
- Consider existing transport patterns and plugin conventions
- Review platform compatibility requirements

#### 3. Code Implementation
- Implement the feature according to JIRA requirements
- Follow existing code patterns and Ruby conventions
- Ensure proper error handling and logging
- Add appropriate documentation and comments
- Consider cross-platform compatibility (Unix, Windows, etc.)
- Follow Train's plugin architecture when applicable

#### 4. Unit Test Creation
- **MANDATORY**: Create comprehensive unit test cases for all new code
- Use Minitest framework (primary testing framework in this repo)
- Ensure test coverage is **> 80%** for the repository
- Follow existing test patterns in `test/unit/` directories
- Mock external dependencies appropriately using Mocha
- Test both success and failure scenarios
- Include platform-specific tests when applicable
- Test transport-specific functionality thoroughly

#### 5. Test Execution & Validation
- Run all unit tests to ensure they pass
- Verify test coverage meets the 80% threshold
- Fix any failing tests or coverage issues
- Ensure no existing tests are broken by changes
- Run integration tests when applicable
- Test on multiple platforms if transport changes are involved

#### 6. Code Quality & Linting
- **MANDATORY**: Run ChefStyle linting before creating PR
- Execute `chefstyle` to check for style and formatting issues
- Run `chefstyle -a` to automatically fix correctable violations

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [inspec/train](https://github.com/inspec/train) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
