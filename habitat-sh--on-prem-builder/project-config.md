---
trigger: always_on
description: This repository contains the Chef Habitat On-Prem Builder, which allows you to host your own private Habitat packages and manage your own origins on-premises.
---

# GitHub Copilot Instructions for on-prem-builder

## Repository Overview

This repository contains the Chef Habitat On-Prem Builder, which allows you to host your own private Habitat packages and manage your own origins on-premises.

### Repository Structure

```
on-prem-builder/
├── .github/                          # GitHub workflows and configurations
│   └── workflows/                    # CI/CD pipeline definitions
├── docs-chef-io/                     # Hugo-based documentation
│   ├── config.toml                   # Hugo configuration
│   ├── go.mod                        # Go module for docs
│   ├── content/habitat/builder/on_prem/  # Documentation content
│   │   ├── _index.md                 # Main documentation index
│   │   ├── troubleshooting.md        # Troubleshooting guide
│   │   ├── configure/                # Configuration guides
│   │   ├── install/                  # Installation guides
│   │   ├── manage/                   # Management guides
│   │   ├── origins/                  # Origin management docs
│   │   └── packages/                 # Package management docs
│   └── static/images/                # Static assets for documentation
├── package_seed_lists/               # Package seed lists for bootstrapping
│   ├── README.md                     # Seed lists documentation
│   └── *.stable                      # Various package seed files
├── pkg-sync/                         # Package synchronization tool
│   ├── main.go                       # Go source code
│   ├── plan.sh                       # Habitat plan file
│   └── README.md                     # Tool documentation
├── pkg-tool/                         # Package analysis tool
│   ├── main.go                       # Go source code
│   ├── plan.sh                       # Habitat plan file
│   ├── build.sh                      # Build script
│   └── README.md                     # Tool documentation
├── scripts/                          # Installation and utility scripts
│   ├── hab-sup.service.sh            # Habitat supervisor service
│   ├── install-hab.sh                # Habitat installation script
│   ├── on-prem-archive.sh            # Archive creation script
│   └── provision.sh                  # Provisioning script
├── terraform/                        # Infrastructure as Code
│   ├── README.md                     # Terraform documentation
│   ├── aws/                          # AWS deployment templates
│   ├── digitalocean/                 # DigitalOcean deployment templates
│   └── templates/                    # Common templates
├── bldr.env.sample                   # Sample environment configuration
├── install.sh                        # Main installation script
├── uninstall.sh                      # Uninstallation script
├── README.md                         # Main project documentation
├── CHANGELOG.md                      # Change log
├── LICENSE                           # License information
└── VERSION                           # Current version
```

## Critical Instructions

### File Modification Restrictions

**IMPORTANT**: Do not modify `*.codegen.go` files if they are present in the repository. These files are automatically generated and manual modifications will be overwritten.

### Testing Requirements

- **Unit Test Coverage**: Always create comprehensive unit test cases for any implementation
- **Coverage Threshold**: The test coverage of the repository must always be maintained above 80%
- **Test Files**: Place test files adjacent to the source files following Go conventions (e.g., `main_test.go` for `main.go`)
- **Test Commands**: Use `go test -v ./...` for running all tests and `go test -cover ./...` for coverage reports

### MCP Server Integration

When a Jira ID is provided in the task:
1. Use the atlassian-mcp-server MCP server to fetch Jira issue details
2. Read the story description, acceptance criteria, and requirements carefully
3. Implement the task according to the specifications in the Jira ticket
4. Ensure all requirements and edge cases mentioned in the ticket are addressed

### Workflow Requirements

All tasks must be performed in a prompt-based manner:
1. **Step-by-step execution**: Complete one step at a time
2. **Progress reporting**: After each step, provide a summary of what was completed
3. **Next step preview**: Clearly state what the next step will be
4. **Remaining tasks**: List all remaining steps to be completed
5. **User confirmation**: Ask if the user wants to continue with the next step before proceeding

## Pull Request Creation Workflow

When prompted to create a PR for changes:

1. **Branch Creation**: Create a new branch using the Jira ID as the branch name
   ```bash
   git checkout -b <JIRA-ID>
   ```

2. **Commit Changes**: Stage and commit all changes with a descriptive message
   ```bash
   git add .
   git commit -m "feat: <brief description of changes> - <JIRA-ID>"
   ```

3. **Push Branch**: Push the changes to the remote repository
   ```bash
   git push origin <JIRA-ID>
   ```

4. **Create PR**: Use GitHub CLI to create the pull request
   ```bash
   gh pr create --title "<JIRA-ID>: <descriptive title>" --body "<PR_DESCRIPTION>" --head <JIRA-ID>

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [habitat-sh/on-prem-builder](https://github.com/habitat-sh/on-prem-builder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
