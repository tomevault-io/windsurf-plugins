---
trigger: always_on
description: Cookstyle is a code linting tool for Chef Infra cookbooks and InSpec profiles, providing style, syntax, logic, and security checks with autocorrection. It is powered by RuboCop and tailored for Chef ecosystem development.
---

# GitHub Copilot Instructions for cookstyle

---

## 1. Repository Analysis & Structure

### Project Purpose
Cookstyle is a code linting tool for Chef Infra cookbooks and InSpec profiles, providing style, syntax, logic, and security checks with autocorrection. It is powered by RuboCop and tailored for Chef ecosystem development.

### Folder Structure Diagram

```
/ (root)
├── bin/                # Executable scripts (cookstyle, cookstyle-profile)
├── config/             # Linting configuration files (chefstyle.yml, cookstyle.yml, default.yml)
├── docs/               # Markdown documentation for cops and usage
├── docs-chef-io/       # Go module for documentation site integration
│   └── assets/cookstyle/ # YAML docs for individual cops
├── habitat/            # Habitat packaging and test scripts
│   └── tests/          # Habitat test scripts
├── lib/                # Main Ruby source code (cookstyle, rubocop, cops, helpers)
│   └── rubocop/
│       ├── chef/       # Chef-specific cops and helpers
│       ├── cop/        # Cop definitions
│       └── monkey_patches/ # RuboCop monkey patches
├── spec/               # RSpec unit tests (mirrors lib/ structure)
│   └── shared/         # Shared test helpers
├── tasks/              # Rake tasks for docs, profiling, spellcheck
├── .expeditor/         # Expeditor build automation config
├── .github/            # GitHub workflows, CODEOWNERS, and Copilot instructions
├── CHANGELOG.md        # Changelog
├── CODE_OF_CONDUCT.md  # Code of conduct
├── CONTRIBUTING.md     # Contribution guide
├── DEVELOPER_GUIDE.md  # Developer guide for maintainers
├── Gemfile             # Ruby gem dependencies
├── LICENSE             # Apache 2.0 license
├── Rakefile            # Rake build/test tasks
├── README.md           # Project overview and usage
├── RELEASE_NOTES.md    # Release notes
├── VERSION             # Current version
├── WRITING_RULES.md    # Guide for writing new cops/rules
```

### Languages, Frameworks, and Technologies
- **Ruby** (primary, for linting engine, cops, and CLI)
- **Go** (for docs-chef-io integration)
- **RSpec** (unit testing)
- **Rake** (build/test tasks)
- **Habitat** (packaging)
- **Expeditor** (build/release automation)

### Modification Guidelines
- **Safe to Modify:**
  - `lib/`, `spec/`, `docs/`, `docs-chef-io/assets/cookstyle/`, `tasks/`, `bin/`, `config/cookstyle.yml`
- **Prohibited/Generated:**
  - `config/chefstyle.yml` (internal, do not edit unless core Chef dev)
  - `config/default.yml` (do not edit unless updating RuboCop engine)
  - `docs-chef-io/go.mod` (managed by docs site tooling)
  - Any files in `vendor/` or `files/` (excluded by config)
- **Never Modify:**
  - LICENSE, CODE_OF_CONDUCT.md, unless updating legal/compliance
  - .expeditor/config.yml, unless updating build automation

### Code Generation Patterns
- All cops must be defined in `config/cookstyle.yml` and have documentation in `docs/` and `docs-chef-io/assets/cookstyle/`.
- Do not modify files in `config/chefstyle.yml` or `config/default.yml` unless you are a core maintainer.

---

## 2. Development Workflow Integration

### Jira Integration (atlassian-mcp-server)
- When a Jira ID is provided, fetch issue details using the MCP server.
- Read the story, analyze requirements, and plan implementation.
- **Workflow Phases:**
  - **Phase 1: Initial Setup & Analysis**
    - Fetch Jira details, analyze repo, plan implementation.
    - Prompt: "Jira story <ID> loaded. Analysis complete. Ready to plan implementation. Proceed?"
  - **Phase 2: Implementation Phase**
    - Implement code, update docs, follow repo structure.
    - Prompt: "Implementation complete. Ready for testing phase. Proceed?"
  - **Phase 3: Testing Phase**
    - Create/extend unit tests, run tests, validate coverage.
    - Prompt: "Testing complete. Ready for PR creation. Proceed?"
  - **Phase 4: Pull Request Creation**
    - Use GH CLI for all git operations, create PR, add labels.
    - Prompt: "PR created. Ready for review. Proceed?"
- **Approval Gates:**
  - After each phase, summarize work, list next steps, and ask for confirmation before proceeding.

---

## 3. Testing Requirements (**Critical - Hard Requirement**)
- **MANDATORY:** All code changes must include comprehensive unit tests.
- **Test Coverage:** >80% coverage is a **hard, non-negotiable requirement**. PRs below this threshold will be rejected.
- **Framework:** Use RSpec for Ruby code. Place tests in `spec/` mirroring the `lib/` structure.
- **Test Structure Example:**
  ```ruby
  # spec/rubocop/cop/chef/my_cop_spec.rb
  require 'spec_helper'
  describe RuboCop::Cop::Chef::MyCop do
    it 'registers an offense for bad code' do
      expect_offense(<<~RUBY)
        ...
      RUBY
    end
    it 'does not register an offense for good code' do
      expect_no_offenses(<<~RUBY)
        ...
      RUBY
    end
  end
  ```
- **Coverage Verification:**
  - Run: `bundle exec rake coverage`
  - Ensure output shows >80% coverage.
- **Test Both:**
  - Positive and negative scenarios
  - Edge cases and error conditions
  - Use mocks for external dependencies

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chef/cookstyle](https://github.com/chef/cookstyle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
