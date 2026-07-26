---
trigger: always_on
description: Ohai is a system information detection tool used by Chef Infra. It profiles your operating system and emits JSON data about system attributes. This repository is an **Active** project under the Chef Infra umbrella.
---

# GitHub Copilot Instructions for Ohai

## Repository Overview

Ohai is a system information detection tool used by Chef Infra. It profiles your operating system and emits JSON data about system attributes. This repository is an **Active** project under the Chef Infra umbrella.

### Repository Structure

```
ohai/
├── .buildkite/                # BuildKite CI configuration
├── .expeditor/                 # Expeditor build system configs
│   ├── config.yml             # Main expeditor configuration
│   ├── buildkite/             # BuildKite pipeline definitions
│   └── *.sh                   # Build and update scripts
├── .github/                   # GitHub configurations
│   ├── CODEOWNERS             # Code ownership definitions
│   ├── workflows/             # GitHub Actions workflows
│   ├── ISSUE_TEMPLATE/        # Issue templates
│   └── dependabot.yml         # Dependabot configuration
├── bin/ohai                   # Main executable
├── lib/                       # Main library code
│   ├── ohai.rb               # Main entry point
│   └── ohai/                 # Core modules
│       ├── application.rb     # CLI application logic
│       ├── config.rb          # Configuration management
│       ├── dsl.rb            # Domain Specific Language
│       ├── loader.rb         # Plugin loading system
│       ├── runner.rb         # Plugin execution engine
│       ├── system.rb         # Main system interface
│       ├── common/           # Common utilities
│       ├── dsl/              # DSL components
│       ├── mixin/            # Mixins for plugins
│       ├── plugins/          # System detection plugins
│       └── util/             # Utility modules
├── spec/                      # Test suite
│   ├── unit/                 # Unit tests
│   ├── functional/           # Functional tests
│   ├── support/              # Test helpers
│   └── data/                 # Test data and fixtures
├── habitat/                   # Habitat packaging
├── tasks/                     # Rake tasks
├── vendor/                    # Vendored dependencies
├── Rakefile                   # Build tasks
├── ohai.gemspec              # Gem specification
├── Gemfile                   # Ruby dependencies
├── README.md                 # Project documentation
├── CONTRIBUTING.md           # Contribution guidelines
├── CHANGELOG.md              # Version history
└── LICENSE                   # Apache 2.0 license
```

## Workflow for Task Implementation

### 1. Task Analysis and Setup
When provided with a Jira ID, follow this workflow:

1. **Fetch Jira Issue Details**
   ```
   Use the atlassian-mcp-server to fetch issue details
   Read and understand the story requirements
   Identify affected components and scope
   ```

2. **Branch Creation**
   ```bash
   # Create feature branch using Jira ID
   git checkout -b JIRA-ID
   ```

3. **Prompt for Continuation**
   - Provide summary of understanding
   - Ask: "Ready to proceed with implementation? What would you like to implement first?"
   - List remaining steps in the workflow

### 2. Implementation Phase

1. **Code Implementation**
   - Implement the required functionality
   - Follow Ruby style guidelines (uses Cookstyle)
   - Ensure code follows existing patterns in the repository
   - Add appropriate error handling and logging

2. **Unit Test Creation**
   - Create comprehensive unit tests using RSpec
   - Place tests in `spec/unit/` following existing structure
   - Ensure test coverage > 80%
   - Use existing test helpers in `spec/support/`
   - Follow existing test patterns (see `spec/spec_helper.rb`)

3. **Prompt for Continuation**
   - Summary: "Implementation completed with unit tests"
   - Next step: "Ready to run tests and validate coverage?"
   - Remaining: "Style checks, functional tests, PR creation"

### 3. Validation Phase

1. **Run Test Suite**
   ```bash
   bundle exec rake spec          # Run all tests
   bundle exec rake style         # Check code style
   bundle exec rake style:auto_correct  # Auto-fix style issues
   ```

2. **Coverage Validation**
   - Ensure test coverage remains > 80%
   - Add additional tests if coverage drops

3. **Prompt for Continuation**
   - Summary: "Tests passing with coverage > 80%"
   - Next step: "Ready to create pull request?"
   - Remaining: "PR creation and submission"

### 4. Pull Request Creation

1. **Commit Changes with DCO Compliance**
   ```bash
   # All commits must include DCO sign-off
   git commit -m "feat: implement JIRA-ID feature

   Detailed description of changes made

   Signed-off-by: Your Name <your.email@example.com>" --signoff
   ```

2. **Push Branch and Create PR**
   ```bash
   # Push the branch
   git push origin JIRA-ID

   # Create PR using GitHub CLI
   gh pr create \
     --title "feat: implement JIRA-ID - Brief Description" \
     --body "$(cat <<EOF
   <h2>Summary</h2>
   <p>Brief description of changes implemented</p>

   <h2>Changes Made</h2>
   <ul>
     <li>Change 1</li>
     <li>Change 2</li>
     <li>Change 3</li>
   </ul>

   <h2>Testing</h2>
   <ul>
     <li>Unit tests added with coverage > 80%</li>

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chef/ohai](https://github.com/chef/ohai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
