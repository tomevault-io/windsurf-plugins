---
trigger: always_on
description: **mixlib-shellout** is a Ruby library that provides a simplified interface to shelling out while collecting both standard out and standard error, and providing full control over environment, working directory, uid, gid, etc. It's part of the Chef ecosystem and provides cross-platform command execution capabilities.
---

# GitHub Copilot Instructions for mixlib-shellout

## Repository Overview

**mixlib-shellout** is a Ruby library that provides a simplified interface to shelling out while collecting both standard out and standard error, and providing full control over environment, working directory, uid, gid, etc. It's part of the Chef ecosystem and provides cross-platform command execution capabilities.

## Repository Structure

```
mixlib-shellout/
├── .expeditor/                    # Expeditor CI/CD configuration
│   ├── config.yml                # Main Expeditor configuration
│   ├── verify.pipeline.yml       # Build pipeline definition
│   ├── run_linux_tests.sh        # Linux test runner
│   ├── run_windows_tests.ps1     # Windows test runner
│   └── update_version.sh         # Version update script
├── .github/
│   ├── CODEOWNERS               # Code ownership definitions
│   ├── ISSUE_TEMPLATE/          # Issue templates
│   ├── workflows/               # GitHub Actions workflows
│   │   └── ci-main-pull-request-checks.yml
│   └── copilot-instructions.md  # This file
├── lib/mixlib/
│   ├── shellout.rb             # Main ShellOut class
│   └── shellout/
│       ├── exceptions.rb       # Custom exceptions
│       ├── helper.rb          # Helper utilities
│       ├── unix.rb           # Unix-specific implementation
│       ├── version.rb        # Version information
│       ├── windows.rb        # Windows-specific implementation
│       └── windows/
│           └── core_ext.rb   # Windows core extensions
├── spec/                      # Test suite
│   ├── spec_helper.rb        # RSpec configuration
│   ├── mixlib/
│   │   ├── shellout_spec.rb  # Main test file
│   │   └── shellout/
│   │       ├── helper_spec.rb    # Helper tests
│   │       └── windows_spec.rb   # Windows-specific tests
│   └── support/              # Test support files
├── vendor/bundle/            # Bundled gems (gitignored in production)
├── Gemfile                   # Ruby dependencies
├── Rakefile                  # Build tasks
├── mixlib-shellout.gemspec   # Gem specification
├── VERSION                   # Version file
├── README.md                 # Project documentation
├── CHANGELOG.md              # Change history
├── CONTRIBUTING.md           # Contribution guidelines
├── CODE_OF_CONDUCT.md        # Code of conduct
└── LICENSE                   # Apache 2.0 license
```

## Jira Integration Workflow

When a Jira ID is provided:

1. **Use the Atlassian MCP Server** to fetch issue details:
   ```bash
   # The atlassian-mcp-server should be configured and available
   # Use the MCP tools to fetch Jira issue information
   ```

2. **Read and analyze the Jira story** to understand:
   - Requirements and acceptance criteria
   - Technical specifications
   - Expected behavior changes
   - Impact on existing functionality

3. **Implement the task** following the story requirements:
   - Make necessary code changes in `lib/mixlib/shellout/` 
   - Ensure cross-platform compatibility (Unix/Windows)
   - Follow existing code patterns and style

## Testing Requirements

### Coverage Standards
- **Maintain > 80% test coverage** at all times
- Run coverage reports using: `bundle exec rspec --format documentation`
- All new features must include comprehensive test cases

### Test Structure
- **Unit tests**: Place in `spec/mixlib/shellout_spec.rb` or appropriate subdirectory
- **Platform-specific tests**: Use `spec/mixlib/shellout/windows_spec.rb` for Windows-only features
- **Helper tests**: Place helper function tests in `spec/mixlib/shellout/helper_spec.rb`

### Test Categories
Use RSpec filters appropriately:
- `:windows_only` - Windows-specific functionality
- `:unix_only` - Unix/Linux-specific functionality  
- `:linux_only` - Linux-specific functionality
- `:requires_root` - Tests requiring root privileges
- `:external` - Tests requiring external dependencies

### Running Tests
```bash
# Run all tests
bundle exec rake spec

# Run specific platform tests
bundle exec rspec --tag unix_only
bundle exec rspec --tag windows_only

# Run with coverage
bundle exec rspec --format documentation
```

## DCO Compliance Requirements

All commits must include a **Developer Certificate of Origin (DCO) sign-off**:

```bash
git commit -s -m "Your commit message"

# Or add manually to commit message:
Signed-off-by: Your Name <your.email@example.com>
```

**DCO Requirements:**
- Every commit must be signed off
- Use your real name (no pseudonyms)
- Use your actual email address
- Indicates you have the right to contribute the code
- Confirms you agree to the DCO terms

## Build System Integration

### Expeditor Configuration
The repository uses **Expeditor** for automated CI/CD:

- **Main config**: `.expeditor/config.yml`
- **Build pipeline**: `.expeditor/verify.pipeline.yml`
- **Notifications**: Sent to `#chef-found-notify` Slack channel
- **Auto-versioning**: Supports major/minor version bumps via labels

### GitHub Actions
- **Workflow**: `.github/workflows/ci-main-pull-request-checks.yml`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chef/mixlib-shellout](https://github.com/chef/mixlib-shellout) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
