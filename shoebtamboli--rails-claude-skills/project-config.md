---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Rails Claude Skills is a Rails generator gem that scaffolds Claude AI skills and agents for Rails projects. It brings Rails generator conventions to Claude AI, making AI-assisted development reusable and distributable across teams and projects.

## Development Commands

### Testing
```bash
# Run all tests
rake spec

# Run tests with RSpec directly
bundle exec rspec

# Run specific test file
bundle exec rspec spec/path/to/file_spec.rb
```

### Code Quality
```bash
# Run RuboCop linter
bundle exec rubocop

# Auto-fix RuboCop issues
bundle exec rubocop -a
```

### Build and Install
```bash
# Build the gem
bundle exec rake build

# Install gem locally
bundle exec rake install

# Build and push to RubyGems (maintainers only)
bundle exec rake release
```

### Setup
```bash
# Initial setup after cloning
bin/setup
```

## CI/CD Pipeline

### GitHub Actions Workflows

**CI Workflow** (`.github/workflows/ci.yml`):
- Triggered on push to `main` and pull requests
- Runs RuboCop linting
- Tests against Ruby versions: 3.0, 3.1, 3.2, 3.3
- Tests against Rails versions: 7.0, 7.1, 7.2
- Builds gem artifact
- Matrix testing ensures compatibility across versions

**Release Workflow** (`.github/workflows/release.yml`):
- Triggered on version tags (e.g., `v0.2.0`)
- Builds and publishes gem to RubyGems
- Creates GitHub release with release notes
- Requires `RUBYGEMS_API_KEY` secret in repository settings

**Dependabot** (`.github/dependabot.yml`):
- Weekly updates for Bundler dependencies
- Weekly updates for GitHub Actions
- Groups RuboCop updates together

### Running CI Locally

```bash
# Run the same checks as CI
bundle exec rspec
bundle exec rubocop

# Test specific Rails version
RAILS_VERSION=7.2 bundle update
RAILS_VERSION=7.2 bundle exec rspec
```

### Release Process

1. Update version in `lib/rails_claude_skills/version.rb`
2. Update `CHANGELOG.md` with release notes
3. Commit changes: `git commit -am "Release v0.x.x"`
4. Create and push tag: `git tag v0.x.x && git push origin v0.x.x`
5. GitHub Actions will automatically build and publish the gem

## Architecture

### Core Components

1. **Module Structure** (`lib/rails_claude_skills.rb`)
   - Main module with configuration support
   - `RailsClaudeSkills.configure` block for global settings
   - Default paths: `.claude/skills` and `.claude/agents`
   - Default model: "sonnet"

2. **Generators** (`lib/generators/claude/`)
   - **InstallGenerator**: Creates `.claude/` directory with preset bundles (basic, fullstack, api)
   - **SkillGenerator**: Creates custom skills with templates (generic, model, controller, frontend)
   - **AgentGenerator**: Creates agents that combine multiple skills
   - **CommandGenerator**: Creates custom Claude Code commands (slash commands)
   - **RuleGenerator**: Creates project-specific rules with templates (generic, testing, security, performance)
   - **ViewsGenerator**: Copies skills, commands, or rules from gem to project for customization

3. **Skills Library** (`lib/generators/claude/skills_library/`)
   - Pre-built skills stored as directories with `SKILL.md` and optional `references/` or `templates/`
   - **Rails Core**: rails-models, rails-controllers, rails-views, rails-api-controllers, rails-hotwire
   - **Authentication & Authorization**: rails-auth-with-devise, rails-authorization-cancancan
   - **Frontend**: tailwindcss
   - **Background Jobs & Email**: rails-jobs, rails-mailers
   - **Testing**: rspec-testing, minitest-testing
   - **Utilities**: rails-debugging, rails-pagination-kaminari, rails-deployment
   - **Planning & Organization**: plan-feature, refine-requirements, create-task-files

4. **Commands Library** (`lib/generators/claude/commands_library/`)
   - Pre-built commands stored as markdown files with YAML frontmatter
   - Available commands: quality, turbo-feature, dbchange, stimulus, create-pr
   - Commands have: description, argument-hint, allowed-tools

5. **Rules Library** (`lib/generators/claude/rules_library/`)
   - Pre-built rules stored as markdown files with optional YAML frontmatter
   - Available rules: code-style, testing, security, database, hotwire
   - Rules can be scoped to specific paths using frontmatter

6. **Railtie Integration** (`lib/rails_claude_skills/railtie.rb`)
   - Integrates gem with Rails application lifecycle
   - Auto-loads generators when gem is present

### Generator Presets

**Basic Preset**:
- Skills: rails-models, rails-controllers, rails-views
- Commands: dbchange
- Rules: code-style, testing, database

**Fullstack Preset**:
- Skills: Basic + rails-hotwire, tailwindcss, rspec-testing
- Commands: Basic + quality, turbo-feature, stimulus, create-pr
- Rules: Basic + hotwire, security

**API Preset**:
- Skills: rails-models, rails-api-controllers, rails-serializers, rails-authentication
- Commands: dbchange, quality
- Rules: code-style, testing, security, database

### Resource Structures

#### Skills
Each skill in the library follows this pattern:
```
skill-name/
├── SKILL.md          # Main skill content with frontmatter (name, description, version)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Shoebtamboli/rails_claude_skills](https://github.com/Shoebtamboli/rails_claude_skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
