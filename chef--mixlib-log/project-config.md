---
trigger: always_on
description: The mixlib-log repository provides a Ruby mixin for enabling a class-based logger object. This is a Ruby gem project that follows Chef's development standards and practices.
---

# Copilot Instructions for mixlib-log Repository

## Repository Overview

The mixlib-log repository provides a Ruby mixin for enabling a class-based logger object. This is a Ruby gem project that follows Chef's development standards and practices.

### Repository Structure

```
mixlib-log/
├── .github/                    # GitHub configuration and workflows
│   ├── CODEOWNERS             # Code ownership configuration
│   ├── ISSUE_TEMPLATE/        # Issue templates
│   ├── dependabot.yml         # Dependabot configuration
│   └── workflows/             # GitHub Actions workflows
├── features/                   # Cucumber features for behavior testing
│   ├── log.feature           # Main logging feature tests
│   ├── steps/                # Step definitions
│   └── support/              # Test support files
├── lib/                       # Main library code
│   └── mixlib/
│       ├── log.rb            # Main logging module
│       └── log/              # Supporting logging classes
│           ├── child.rb      # Child logger implementation
│           ├── formatter.rb  # Log formatting
│           ├── logger.rb     # Logger class
│           ├── logging.rb    # Logging utilities
│           └── version.rb    # Version information
├── spec/                      # RSpec unit tests
│   ├── spec_helper.rb        # Test configuration
│   └── mixlib/               # Test files mirroring lib structure
├── vendor/                    # Bundled dependencies
├── Gemfile                    # Gem dependencies
├── Gemfile.lock              # Locked dependency versions
├── Rakefile                   # Build tasks
├── mixlib-log.gemspec        # Gem specification
├── README.md                 # Project documentation
├── CHANGELOG.md              # Change history
├── VERSION                   # Version file
└── LICENSE                   # Apache 2.0 license
```

## Jira Integration with MCP Server

When a Jira ID is provided:

1. **Use the atlassian-mcp-server** to fetch Jira issue details
2. **Read the story requirements** thoroughly from the Jira issue
3. **Implement the task** according to the specifications
4. **Reference the Jira ID** in commits and PR descriptions

### Jira Workflow Commands
```bash
# Example MCP server usage for Jira integration
# The atlassian-mcp-server should be configured to access your Jira instance
# Use the following pattern when Jira ID is provided:
# 1. Fetch issue: mcp_atlassian-mcp_getJiraIssue with cloudId and issueIdOrKey
# 2. Read requirements from the issue description and acceptance criteria
# 3. Implement according to the story requirements
```

## Testing Requirements

### Unit Test Coverage
- **Maintain >80% test coverage** for all new and modified code
- Use RSpec for unit tests (located in `spec/` directory)
- Use Cucumber for integration/behavior tests (located in `features/` directory)
- Run tests with: `bundle exec rake spec` and `bundle exec rake features`

### Test Commands
```bash
# Run all tests
bundle exec rake

# Run only unit tests
bundle exec rake spec

# Run only integration tests  
bundle exec rake features

# Run style checks
bundle exec rake style

# Check coverage
bundle exec rspec --format html --out coverage/index.html
```

## Pull Request and Branch Management

### Branch Creation and PR Workflow

When prompted to create a PR for changes:

1. **Create a branch** using the Jira ID as the branch name
2. **Push changes** to the new branch
3. **Create a PR** using GitHub CLI
4. **Include HTML-formatted description** with summary of changes

### GitHub CLI Commands
```bash
# Create and switch to new branch (use Jira ID as branch name)
git checkout -b JIRA-123

# Stage and commit changes with DCO sign-off
git add .
git commit -s -m "feat: implement feature for JIRA-123

Detailed description of changes made.

Signed-off-by: Your Name <your.email@example.com>"

# Push branch
git push origin JIRA-123

# Create PR with HTML description
gh pr create --title "feat: implement feature for JIRA-123" \
  --body "<h2>Summary</h2>
<p>Brief description of changes</p>
<h2>Changes Made</h2>
<ul>
<li>Change 1</li>
<li>Change 2</li>
</ul>
<h2>Testing</h2>
<p>Description of testing performed</p>
<h2>Jira Issue</h2>
<p>Resolves: JIRA-123</p>"
```

**Important**: All tasks are performed on the local repository. Do not reference `~/.profile` for GitHub authentication steps.

## DCO Compliance Requirements

All commits **MUST** be signed off to comply with the Developer Certificate of Origin (DCO):

### DCO Sign-off Requirements
- **Every commit** must include a `Signed-off-by` line
- Use `git commit -s` to automatically add sign-off
- Sign-off certifies you have the right to submit the code under the project's license
- Format: `Signed-off-by: Your Name <your.email@example.com>`

### DCO Commands
```bash
# Commit with automatic DCO sign-off
git commit -s -m "your commit message"

# Add DCO sign-off to existing commit
git commit --amend -s

# Check if commits are signed
git log --show-signature
```

## Build System Integration

### GitHub Actions Workflows
The repository uses GitHub Actions for CI/CD with the following workflow:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chef/mixlib-log](https://github.com/chef/mixlib-log) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
