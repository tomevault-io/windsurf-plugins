---
trigger: always_on
description: **Mixlib::Config** is a Ruby library that provides a class-based configuration object system used throughout the Chef ecosystem. This repository maintains the core configuration management functionality that allows applications to define and manage configuration options with support for default values, nested contexts, strict mode validation, and multiple file format inputs (Ruby, YAML, JSON, TOML).
---

# Copilot Instructions for Mixlib::Config Repository

## Repository Overview

**Mixlib::Config** is a Ruby library that provides a class-based configuration object system used throughout the Chef ecosystem. This repository maintains the core configuration management functionality that allows applications to define and manage configuration options with support for default values, nested contexts, strict mode validation, and multiple file format inputs (Ruby, YAML, JSON, TOML).

### Repository Structure

```
mixlib-config/
├── .expeditor/                   # Expeditor CI/CD configuration
│   ├── config.yml               # Main Expeditor configuration
│   ├── run_linux_tests.sh       # Linux test execution script
│   ├── run_windows_tests.ps1    # Windows test execution script
│   ├── update_version.sh        # Version update automation
│   └── verify.pipeline.yml      # Verification pipeline
├── .github/                     # GitHub configuration and templates
│   ├── CODEOWNERS              # Code ownership definitions
│   ├── ISSUE_TEMPLATE/         # Issue templates
│   ├── dependabot.yml         # Dependency update configuration
│   └── workflows/              # GitHub Actions workflows
│       └── ci-main-pull-request-checks.yml
├── features/                    # Cucumber feature tests
│   ├── mixlib_config.feature   # Main feature specifications
│   ├── step_definitions/       # Cucumber step definitions
│   ├── steps/                  # Additional test steps
│   └── support/                # Test support files
├── lib/                        # Main library source code
│   └── mixlib/
│       ├── config.rb           # Main configuration class
│       └── config/             # Configuration modules
│           ├── configurable.rb # Configurable mixin
│           ├── reopened_config_context_with_configurable_error.rb
│           ├── reopened_configurable_with_config_context_error.rb
│           ├── unknown_config_option_error.rb
│           └── version.rb      # Version information
├── spec/                       # RSpec unit tests
│   ├── spec_helper.rb         # RSpec configuration
│   └── mixlib/
│       └── config_spec.rb     # Main configuration tests
├── CHANGELOG.md               # Release notes and changes
├── CODE_OF_CONDUCT.md         # Community guidelines
├── CONTRIBUTING.md            # Contribution guidelines
├── Gemfile                    # Ruby dependencies
├── LICENSE                    # Apache 2.0 license
├── mixlib-config.gemspec      # Gem specification
├── NOTICE                     # Legal notices
├── Rakefile                   # Rake build tasks
├── README.md                  # Project documentation
└── VERSION                    # Current version number
```

## Jira Integration Workflow

When a Jira ID is provided in any task or request:

1. **Use the Atlassian MCP Server** to fetch Jira issue details:
   - Use the `mcp_atlassian-mcp_getJiraIssue` tool to retrieve issue information
   - Use the `mcp_atlassian-mcp_search` tool for Rovo Search when needed
   - Read and understand the story requirements, acceptance criteria, and linked issues

2. **Implementation Process**:
   - Analyze the Jira issue requirements thoroughly
   - Identify affected components and files
   - Plan implementation approach based on story requirements
   - Implement changes following the established patterns in the codebase

## Testing Requirements

### Unit Test Coverage
- **Maintain >80% test coverage** at all times
- Run tests using: `bundle exec rspec` or `rake spec`
- Coverage reports should be generated and verified
- All new functionality must include comprehensive unit tests

### Test Structure
- Unit tests are located in `spec/` directory
- Use RSpec testing framework with the existing `spec_helper.rb` configuration
- Follow existing test patterns and naming conventions
- Include both positive and negative test cases
- Test edge cases and error conditions

### Cucumber Features
- Behavioral tests are in `features/` directory using Cucumber
- Update feature files when adding new functionality
- Ensure step definitions match new behaviors

## Pull Request Creation Workflow

When prompted to create a PR for changes:

1. **Branch Management**:
   ```bash
   # Create branch using Jira ID as branch name
   git checkout -b [JIRA-ID]
   
   # Make your changes and commit with DCO compliance
   git add .
   git commit -s -m "feat: [JIRA-ID] Brief description of changes
   
   Detailed description of what was implemented.
   
   Signed-off-by: Your Name <your.email@example.com>"
   ```

2. **Push and Create PR**:
   ```bash
   # Push changes to the branch
   git push origin [JIRA-ID]
   
   # Create PR using GitHub CLI
   gh pr create --title "[JIRA-ID] Brief description" \
     --body "$(cat <<EOF
   <h2>Summary</h2>
   <p>Brief summary of changes made</p>
   
   <h2>Changes Made</h2>
   <ul>
   <li>Detailed list of changes</li>

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chef/mixlib-config](https://github.com/chef/mixlib-config) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
