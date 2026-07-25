---
trigger: always_on
description: This document provides comprehensive instructions for GitHub Copilot when working with the Chef Workstation repository.
---

# GitHub Copilot Instructions for Chef Workstation

This document provides comprehensive instructions for GitHub Copilot when working with the Chef Workstation repository.

## Repository Overview

Chef Workstation is a comprehensive package that installs everything needed to get started with Chef products on Windows, Mac, and Linux. It includes Chef Infra Client, Chef InSpec, Chef Habitat, Chef Command Line Tool, Test Kitchen, Cookstyle, and various plugins.

## Repository Structure

The Chef Workstation repository follows this structure:

```
chef-workstation/
├── .expeditor/                    # Build and release automation
├── .github/                       # GitHub workflows and templates
│   ├── CODEOWNERS                # Code ownership definitions
│   ├── ISSUE_TEMPLATE/           # Issue templates
│   ├── dependabot.yml           # Dependabot configuration
│   └── workflows/               # CI/CD workflows (sonarqube.yml, unit.yml)
├── CHANGELOG.md                  # Version history and changes
├── CODE_OF_CONDUCT.md           # Community guidelines
├── CONTRIBUTING.md              # Contribution guidelines
├── README.md                    # Project documentation
├── RELEASE_PROCESS.md           # Release process documentation
├── Gemfile                      # Ruby dependencies
├── Rakefile                     # Build tasks
├── VERSION                      # Current version
├── components/                  # Core components
│   ├── chef-automate-collect/   # Chef Automate data collection tool (Go)
│   ├── gems/                    # Ruby gems and dependencies
│   ├── main-chef-wrapper/       # Main chef wrapper (Go)
│   ├── packaging/               # Packaging components (Chocolatey)
│   └── rehash/                  # Rehash utilities
├── coverage/                    # Code coverage reports
├── dev-docs/                    # Developer documentation
│   ├── architecture/            # Architecture diagrams and docs
│   ├── clibuddy/               # CLI buddy configurations
│   └── img/                    # Documentation images
├── habitat/                     # Chef Habitat packaging
├── omnibus/                     # Omnibus packaging configuration
│   ├── config/                  # Omnibus configuration
│   ├── cookbooks/              # Build cookbooks
│   ├── files/                  # Build files and scripts
│   ├── package-scripts/        # Package installation scripts
│   ├── resources/              # Omnibus resources
│   └── verification/           # Package verification tests
├── test/                       # Test suites
│   └── integration/            # Integration tests
├── sonar-project.properties    # SonarQube configuration
├── cspell.json                # Spell check configuration
├── dobi.yaml                  # Docker build configuration
└── Dockerfile                 # Container build definition
```

## Development Workflow

### 1. Task Implementation with Jira Integration

When a Jira ID is provided:

1. **Fetch Jira Details**: Use the `atlassian-mcp-server` MCP server to fetch the Jira issue details
2. **Read and Analyze**: Carefully read the story description, acceptance criteria, and requirements
3. **Plan Implementation**: Break down the task into actionable steps
4. **Implement**: Follow the implementation guidelines below

### 2. Implementation Guidelines

- **Code Quality**: Follow existing code patterns and conventions
- **Testing**: Always create comprehensive unit tests for your implementation
- **Coverage**: Maintain code coverage above 80% for the repository
- **Documentation**: Update relevant documentation when making changes
- **Dependencies**: Use appropriate dependency management (Gemfile for Ruby, go.mod for Go)

### 3. Testing Requirements

**Comprehensive Testing Strategy:**
- Write unit tests for all new functionality
- Ensure integration tests pass
- Maintain overall repository coverage > 80% (this is a hard requirement)
- Run existing test suites to ensure no regressions
- Test both positive and negative scenarios
- Create mock objects for external dependencies
- Use table-driven test patterns where appropriate

**Testing Framework Usage:**
- Ruby: RSpec, Minitest (~> 5.16)
- Go: Built-in testing package with cross-platform considerations
- Integration: Test Kitchen with multiple drivers

**Test Structure Guidelines:**
```ruby
# Ruby test example
describe "ClassName" do
  context "when condition" do
    it "should behave correctly" do
      # Setup
      # Execute
      # Assert
    end
  end
  
  context "when error condition" do
    it "should handle errors gracefully" do
      # Test error scenarios
    end
  end
end
```

```go
// Go test example
func TestServiceMethod(t *testing.T) {
    tests := []struct {
        name     string
        input    interface{}
        expected interface{}
        wantErr  bool
    }{
        // Test cases here
    }
    
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            // Test implementation
        })
    }
}
```

**Coverage Verification:**
- Generate and verify coverage reports after running tests
- Ensure all new code has appropriate test coverage
- Test edge cases and error conditions thoroughly

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chef/chef-workstation](https://github.com/chef/chef-workstation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
