---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Badgetizr is a tool that automatically adds customizable badges to GitHub pull requests to increase team productivity. It's distributed as both a Homebrew tap and a GitHub Action.

## Core Architecture

### Main Components
- `badgetizr` - Main bash script that orchestrates badge creation
- `utils.sh` - Helper functions and utilities (defines version, help text)
- `.badgetizr.yml` - Configuration file defining badge types and settings
- `action.yml` - GitHub Action configuration for CI/CD integration
- `Formula/badgetizr.rb` - Homebrew formula for package distribution

### Badge Types Supported
1. **Ticket Badge** - Extracts and links ticket IDs from PR titles using regex patterns
2. **WIP Badge** - Shows when PR title contains "WIP"
3. **Base Branch Badge** - Displays target branch information
4. **CI Badge** - Shows build status with configurable build numbers and URLs
5. **Dynamic Badge** - Custom badges based on PR body content patterns (e.g., task checklists)

### Configuration System
- Uses YAML configuration files (`.badgetizr.yml` by default)
- Each badge type has `enabled` flag and customizable `settings`
- Supports regex pattern matching with `sed_pattern` for dynamic content extraction
- Badge appearance customizable: color, label, logo (from simpleicons.org), URLs

## Common Development Commands

### Running Badgetizr
```bash
# Basic usage with default config
./badgetizr --pr-id=123

# With custom configuration
./badgetizr -c custom.yml --pr-id=123

# With CI integration parameters
./badgetizr --pr-id=123 --pr-destination-branch=master --pr-build-number=456 --pr-build-url="https://..."

# Show version
./badgetizr -v

# Show help
./badgetizr -h
```

### Development Setup
```bash
# Install dependencies (done by configure script)
./configure

# Test locally (requires GITHUB_TOKEN)
export GITHUB_TOKEN="your_token"
./badgetizr --pr-id=123
```

### Publishing (Maintainers Only)
```bash
# Automated release process - updates version everywhere
./publish.sh 1.5.5
```

### Creating Pull Requests (Claude and Contributors)

**IMPORTANT**: When creating pull requests, always use the PR template located at `.github/pull_request_template.md`.

#### For Claude Code (GitHub Workflow)
When Claude creates a PR from an issue, it MUST:
1. Read the PR template: `.github/pull_request_template.md`
2. Use the template structure for the PR body
3. Fill in the checklist items appropriately
4. Include the GitHub Issue ID in the PR title: `[GH-XXX] Clear description`
5. If testing on GitLab was done, include the MR link in the "GitLab Testing" section

Example PR body structure:
```markdown
## Comments
- [Brief description of changes]

## Checklist
- [x] The PR starts by `[GH-43] Add integration tests`
- [ ] I have added WIP to my PR title if needed
- [ ] I have tested on GitLab and added the MR link below

## GitLab Testing
**GitLab MR Link**: [Link if applicable]
```

#### For Manual Contributions
- Always create a feature branch: `feat/GH-XXX_description`
- Use the PR template when creating the PR
- Reference the related issue in the PR title
- Test on both GitHub and GitLab when possible

## Dependencies

### Required Tools
- `gh` (GitHub CLI) - for PR interaction
- `yq` - for YAML configuration parsing
- Standard bash utilities: `sed`, `curl`, `awk`

### Installation Handled By
- **GitHub Action**: Automatically installs via `action.yml` steps
- **Homebrew**: Dependencies declared in `Formula/badgetizr.rb`
- **Manual**: Run `./configure` script

## Key Integration Points

### GitHub Action Usage
```yaml
- name: Run Badgetizr
  uses: aiKrice/homebrew-badgetizr@1.5.4
  with:
    pr_id: ${{ github.event.pull_request.number }}
    configuration: .badgetizr.yml
    pr_destination_branch: ${{ github.event.pull_request.base.ref }}
  env:
    GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### Homebrew Distribution
- Formula located in `Formula/badgetizr.rb`
- Installs symlink from `libexec/badgetizr` to `bin/badgetizr`
- SHA256 checksum required for tarball verification

## Release Process

The `publish.sh` script automates:
1. Version bumping in `utils.sh`, `README.md`, workflow files
2. Git branch management (develop → master)
3. GitHub release creation with auto-generated notes
4. Homebrew formula SHA256 calculation and update
5. Automatic backmerge to develop branch

## Environment Variables

- `GITHUB_TOKEN` or `GH_TOKEN` - Required for GitHub API access
- Configuration passed via command-line arguments to the main script

## Recent Development Notes

### Homebrew Formula Fix (Fixed in v1.5.5)
- **Problem**: `badgetizr` script couldn't find `utils.sh` after Homebrew installation
- **Root Cause**: Homebrew was creating a symlink without setting `UTILS_PATH` environment variable
- **Solution**: Modified `Formula/badgetizr.rb` to use `write_env_script` instead of `install_symlink`
```ruby
# Before (broken)
bin.install_symlink libexec/"badgetizr"

# After (working)
(bin/"badgetizr").write_env_script libexec/"badgetizr", UTILS_PATH: libexec/"utils.sh"
```

### WIP Badge Labelized Feature (Added in v1.6.0)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aiKrice/homebrew-badgetizr](https://github.com/aiKrice/homebrew-badgetizr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-02 -->
