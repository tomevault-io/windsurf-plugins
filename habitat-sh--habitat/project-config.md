---
trigger: always_on
description: Habitat is Chef's application automation framework that builds, deploys, and manages the applications. This repository contains the core Habitat components written primarily in Rust.
---

# GitHub Copilot Instructions for Habitat Repository

## Repository Overview

Habitat is Chef's application automation framework that builds, deploys, and manages the applications. This repository contains the core Habitat components written primarily in Rust.

## Repository Structure

```
habitat/
├── .expeditor/                   # Build and release automation scripts
│   ├── config.yml                # Expeditor configuration
│   └── scripts/                  # Build, test, and release scripts
├── .github/                      # GitHub workflows and templates
├── components/                   # Main Habitat components
│   ├── builder-api-client/       # Builder API client library
│   ├── builder-api/              # Builder API service
│   ├── common/                   # Shared utilities and common code
│   ├── core/                     # Core Habitat functionality
│   ├── hab/                      # Main hab CLI tool
│   ├── launcher/                 # Habitat launcher service
│   ├── pkg-export-container/     # Container export functionality
│   ├── pkg-export-tar/           # Tar export functionality
│   ├── plan-build/               # Plan building utilities
│   ├── studio/                   # Habitat Studio environment
│   └── sup/                      # Habitat Supervisor
├── support/                      # Build support files and scripts
├── test/                         # End-to-end tests
├── tools/                        # Development and build tools
├── Cargo.toml                    # Rust workspace configuration
├── Dockerfile                    # Docker build configuration
├── Justfile                      # Just command runner configuration
├── Makefile                      # Build automation
└── VERSION                       # Current version number
```

### Critical File Modification Rules


**🚨 NEVER MODIFY THESE FILES:**
- `.expeditor/config.yml` - This controls the release pipeline
- Files in `.expeditor/scripts/` unless specifically requested
- `POWERSHELL_VERSION` file - This is managed by the release process
- `RUST_NIGHTLY_VERSION` file - This is managed by the release process
- `RUSTFORMAT_VERSION` file - This is managed by the release process
- `VERSION` file - This is managed by the release process

## Task Implementation Workflow

### 1. Initial Setup and Jira Integration

When a Jira ID is provided:

1. **Fetch Jira Details**: Use the atlassian-mcp-server to fetch issue details
   ```bash
   # Use MCP server to get Jira issue information
   # Read the story description, acceptance criteria, and requirements
   ```

2. **Analyze Requirements**:
   - Read the Jira story thoroughly
   - Understand the scope and acceptance criteria
   - Identify which components need modification

3. **Provide Summary**: Give a clear summary of what needs to be implemented

### 2. Pre-Implementation Analysis

1. **Component Identification**: Determine which Habitat components are affected
2. **Impact Assessment**: Analyze potential breaking changes
3. **Test Strategy**: Plan unit tests and integration tests needed
4. **File Review**: Identify files that will be modified (avoiding prohibited files)

### 3. Implementation Phase

1. **Code Implementation**:
   - Follow Rust best practices and Habitat coding standards
   - Ensure thread safety and error handling
   - Use appropriate logging levels
   - Follow existing patterns in the codebase

2. **Unit Test Creation**:
   - Create comprehensive unit tests for new functionality
   - Ensure test coverage remains > 80%
   - Use existing test patterns and utilities
   - Test error conditions and edge cases

3. **Integration Testing**:
   - Run relevant integration tests
   - Verify compatibility with existing functionality

### 4. Quality Assurance

1. **Code Coverage**: Verify coverage is > 80%
   ```bash
   cargo tarpaulin --out html --output-dir coverage/
   ```

2. **Linting and Formatting**:
   ```bash
   cargo clippy --all-targets --all-features
   cargo fmt --check
   ```

3. **Build Verification**:
   ```bash
   cargo build --all-targets
   cargo test --all
   ```

### 5. Git Workflow, PR Creation, and Description

1. **Branch Creation**: Use Jira ID as branch name
   ```bash
   git checkout -b <JIRA-ID>
   ```

2. **Commit Changes**:
   ```bash
   git add .
   git commit -s -m "<JIRA-ID>: Brief description of changes"
   ```

3. **Push and Create PR**:
   ```bash
   git push origin <JIRA-ID>
   gh pr create --title "<JIRA-ID>: Brief title" --body "<HTML formatted description>" --label "ai-assisted"
   ```

4. **PR Description Format** (HTML):
   ```html
   <h2>Summary</h2>
   <p>Brief description of changes made</p>

   <h2>Changes Made</h2>
   <ul>
     <li>Change 1</li>
     <li>Change 2</li>
   </ul>

   <h2>Testing</h2>
   <ul>
     <li>Unit tests added/updated</li>
     <li>Integration tests verified</li>
     <li>Coverage maintained > 80%</li>
   </ul>

   <h2>AI Assistance</h2>
   <p>This work was completed with AI assistance following Progress AI policies</p>

   <h2>Jira Link</h2>
   <p><a href="link-to-jira-issue">JIRA-ID</a></p>
   ```

### 6. Update JIRA Ticket (MANDATORY)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [habitat-sh/habitat](https://github.com/habitat-sh/habitat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
