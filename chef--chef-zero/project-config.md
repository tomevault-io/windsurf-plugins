---
trigger: always_on
description: Chef Zero is an in-memory Chef server implementation designed for testing and development purposes. It provides a lightweight, fast alternative to a full Chef server for local development and testing scenarios.
---

# Chef Zero - GitHub Copilot Development Instructions

## Repository Overview

Chef Zero is an in-memory Chef server implementation designed for testing and development purposes. It provides a lightweight, fast alternative to a full Chef server for local development and testing scenarios.

## Repository Structure

```
chef-zero/
├── bin/
│   └── chef-zero                    # CLI executable
├── lib/
│   ├── chef_zero.rb                 # Main entry point
│   └── chef_zero/
│       ├── version.rb               # Version management
│       ├── server.rb                # Core server implementation
│       ├── rest_*.rb               # REST API components
│       ├── chef_data/              # Chef data handling
│       │   ├── acl_path.rb
│       │   ├── cookbook_data.rb
│       │   ├── data_normalizer.rb
│       │   └── default_creator.rb
│       ├── data_store/             # Data storage implementations
│       │   ├── memory_store*.rb    # In-memory storage
│       │   ├── raw_file_store.rb   # File-based storage
│       │   └── interface_*.rb      # Storage interfaces
│       ├── endpoints/              # API endpoint implementations
│       │   ├── *_endpoint.rb       # Various Chef API endpoints
│       │   └── cookbooks_base.rb   # Cookbook handling base
│       └── solr/                   # Search functionality
│           └── solr_doc.rb
├── spec/                           # RSpec test suite
│   ├── *_spec.rb                  # Unit tests
│   ├── run_oc_pedant.rb           # Integration test runner
│   └── support/                   # Test support files
├── playground/                     # Test data and examples
│   ├── cookbooks/
│   ├── data_bags/
│   ├── environments/
│   └── nodes/
├── .expeditor/                     # Build and release automation
├── .github/                        # GitHub workflows and templates
└── Rakefile                       # Build tasks and test runners
```

## Development Workflow

### 1. Jira Integration (MCP Server)

When a Jira ID is provided, use the **atlassian-mcp-server** to:

1. Fetch issue details using `mcp_atlassian-mcp_getJiraIssue`
2. Read the story description and acceptance criteria
3. Understand requirements before implementation
4. Update Jira with progress using `mcp_atlassian-mcp_addCommentToJiraIssue`

### 2. Implementation Process

#### Step-by-Step Workflow

1. **Analysis Phase**
   - Fetch and analyze Jira issue details
   - Review existing code and tests
   - Identify files that need modification
   - Plan implementation approach

2. **Development Phase**
   - Create feature branch named after Jira ID
   - Implement changes following Ruby/Chef conventions
   - Ensure code follows existing patterns and styles
   - Add comprehensive error handling

3. **Testing Phase**
   - Write unit tests for new functionality
   - Ensure test coverage remains > 80%
   - Run existing test suite to prevent regressions
   - Test with both RSpec and oc-pedant integration tests

4. **Documentation Phase**
   - Update relevant documentation
   - Add code comments where necessary
   - Update CHANGELOG.md if applicable

5. **Review and PR Phase**
   - Create pull request with proper description
   - Link to Jira issue
   - Ensure all CI checks pass

### 3. Testing Requirements

**Mandatory Testing Standards:**

- Minimum 80% test coverage required
- Unit tests for all new functionality using RSpec
- Integration tests using oc-pedant when applicable
- Test both success and error scenarios
- Mock external dependencies appropriately

**Test Commands:**

```bash
# Run unit tests
rake spec

# Run integration tests
rake pedant

# Run style checks
rake style

# Run all tests with coverage
COVERAGE=true rake spec
```

### 4. DCO Compliance Requirements

**All commits MUST be signed off** according to the Developer Certificate of Origin (DCO):

- Add `-s` flag to all commits: `git commit -s -m "Your commit message"`
- Commits without proper sign-off will be rejected
- Use your real name and email address
- Each commit must include: `Signed-off-by: Your Name <your.email@example.com>`

### 5. Branch Management and PR Creation

When prompted to create a PR:

```bash
# Create and switch to feature branch (use Jira ID as branch name)
gh repo clone chef/chef-zero  # if not already cloned
cd chef-zero
git checkout -b JIRA-123  # Replace with actual Jira ID

# Make your changes and commit with DCO sign-off
git add .
git commit -s -m "feat: implement feature described in JIRA-123"

# Push branch and create PR
git push origin JIRA-123
gh pr create --title "feat: Brief description of changes" \
  --body "$(cat <<EOF
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
<li>Integration tests passing</li>
<li>Code coverage maintained >80%</li>
</ul>

<h2>Related Issues</h2>
<p>Fixes: JIRA-123</p>
EOF
)"
```

### 6. Build System Integration

**Expeditor Build System:**

- Automatic version bumping on merge
- Changelog generation
- Gem publishing to RubyGems
- Available labels for version control:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chef/chef-zero](https://github.com/chef/chef-zero) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
