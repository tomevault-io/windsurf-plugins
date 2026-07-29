---
trigger: always_on
description: Chef Infra is a configuration management tool designed to bring automation to your entire infrastructure. This repository contains the core Chef Infra Client and related utilities.
---

# GitHub Copilot Instructions for Chef Infra Repository

## Repository Overview

Chef Infra is a configuration management tool designed to bring automation to your entire infrastructure. This repository contains the core Chef Infra Client and related utilities.

## Environment Rules
Copilot must always detect the OS and Shell and provide commands specific to that OS and shell.
If the OS is not specified, Copilot must ask the user to specify it before providing any commands.
Copilot should never mix commands across different platforms.

## Repository Structure

The Chef repository follows a modular structure with the following key components:

```
chef/
├── .github/                    # GitHub workflows, templates, and configurations
├── chef-bin/                   # Chef binary executables and CLI tools
├── chef-config/                # Configuration management for Chef
├── chef-utils/                 # Utility functions and helpers
├── distro/                     # Distribution-specific templates and files
├── docs/                       # Documentation and development guides
├── ext/                        # External extensions (e.g., win32-eventlog)
├── habitat/                    # Habitat packaging configuration
├── kitchen-tests/              # Test Kitchen integration tests
├── lib/                        # Main Chef library code
│   ├── chef.rb                 # Main entry point
│   └── chef/                   # Core Chef modules and classes
├── spec/                       # Test specifications
│   ├── unit/                   # Unit tests
│   ├── functional/             # Functional tests
│   ├── integration/            # Integration tests
│   └── support/                # Test support files
├── tasks/                      # Rake tasks for development
└── vendor/                     # Vendored dependencies
```

### Key Files

- `chef.gemspec` - Main gem specification
- `Rakefile` - Build and test tasks
- `Gemfile` - Ruby dependencies
- `CONTRIBUTING.md` - Contribution guidelines
- `spec/spec_helper.rb` - Test configuration

## Workflow for Task Implementation

When implementing tasks in this repository, follow this comprehensive workflow:

### 1. Task Analysis and Planning

- **Jira Integration**: When a Jira ID is provided, use the atlassian-mcp-server to fetch issue details
- **Story Understanding**: Read and analyze the Jira story thoroughly to understand requirements
- **Task Breakdown**: Break down complex tasks into smaller, manageable components
- **Impact Assessment**: Evaluate which files and modules will be affected

### 2. Pre-Implementation Steps

- Analyze the existing codebase structure
- Identify the appropriate modules/files to modify
- Review related existing tests
- Check for any dependencies or prerequisites

### 3. Implementation Phase

- Follow Ruby best practices and Chef coding conventions
- Ensure backward compatibility when possible
- Implement changes incrementally
- Write clean, well-documented code
- Follow the existing code style and patterns

### 4. Testing Requirements

- **Unit Tests**: Create comprehensive unit tests for all new functionality
- **Coverage Target**: Maintain test coverage > 80% for the repository
- **Test Types**: Include unit, functional, and integration tests as appropriate
- **Test Location**: Place tests in appropriate `spec/` subdirectories
- **Test Framework**: Use RSpec for testing

### 5. Validation and Quality Assurance

- Run all existing tests to ensure no regressions
- Verify code quality and adherence to Ruby standards
- Check for any security implications
- Validate functionality across different environments

### 6. Documentation

- Update relevant documentation
- Add inline code comments where necessary
- Update CHANGELOG.md if applicable
- Ensure README updates if public API changes

### 7. Pull Request Creation

When prompted to create a PR:

- Use GitHub CLI (`gh`) for branch and PR operations
- Branch naming:
  - Ask for preferred developer initials the first time encountered
  - If a Jira ID is provided, include it in the branch name
  - For work based on `main`, create branches as `{dev-initials}/{jira-id}-{short-description}` (or `{dev-initials}/{short-description}` when no Jira ID exists)
  - For work based on a release branch (for example `chef-18`), create branches as `{dev-initials}/{release-branch}-{jira-id}-{short-description}` (or `{dev-initials}/{release-branch}-{short-description}` when no Jira ID exists)
- Create commits with DCO sign-off (`git commit -s`)
- Push changes to the new branch
- Create a PR with:
  - **Title**: Clear, descriptive title referencing the Jira ID
  - **Description**: HTML-formatted summary of changes made
- For CVE fixes, add the `CVE` label and prefix the PR description with `[CVE]`
- All operations should be performed on the local repository

## MCP Server Integration

### Atlassian MCP Server Usage

When working with Jira integration:

- Use the `atlassian-mcp-server` MCP server for all Jira operations
- Fetch issue details using the provided Jira ID
- Parse and understand the story requirements
- Reference the Jira ID in commits and PR descriptions


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chef/chef](https://github.com/chef/chef) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
