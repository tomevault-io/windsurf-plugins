---
trigger: always_on
description: This document defines the authoritative operational guide for AI-assisted contributors working in the `mixlib-cli` repository. It standardizes how tasks are clarified, planned, implemented, tested, documented, and delivered—especially when driven by Jira issues and iterative prompt-based collaboration. It enforces compliance (DCO, coverage, protected file safety), codifies branching + PR conventions, and describes integration points (CI, Expeditor, labels, automation). Use this as the single sou
---

## Purpose

This document defines the authoritative operational guide for AI-assisted contributors working in the `mixlib-cli` repository. It standardizes how tasks are clarified, planned, implemented, tested, documented, and delivered—especially when driven by Jira issues and iterative prompt-based collaboration. It enforces compliance (DCO, coverage, protected file safety), codifies branching + PR conventions, and describes integration points (CI, Expeditor, labels, automation). Use this as the single source of truth for workflow execution.

---
## Repository Structure

Concise directory tree (non-vendored, depth-limited). Each line includes a brief purpose.

```
.
├── CHANGELOG.md              # Project changelog (Expeditor manages updates)
├── CODE_OF_CONDUCT.md        # Community conduct policy (protected)
├── CONTRIBUTING.md           # Contribution guidelines (reference only)
├── Gemfile                   # Development dependencies bundler entrypoint
├── LICENSE                   # Apache 2.0 license (protected; never modify)
├── NOTICE                    # Notice file (protected)
├── Rakefile                  # Rake tasks (specs, docs)
├── README.md                 # Project overview and usage
├── VERSION                   # Current gem version (bumped by Expeditor)
├── mixlib-cli.gemspec        # Gem specification & metadata
├── .rubocop.yml              # Cookstyle/Rubocop configuration
├── .expeditor/               # Expeditor automation (release, versioning)
│   ├── config.yml            # Expeditor pipeline + subscription rules
│   ├── verify.pipeline.yml   # Buildkite verification pipeline definition
│   ├── run_linux_tests.sh    # Linux test + bundle caching helper
│   ├── run_windows_tests.ps1 # Windows test runner
│   └── update_version.sh     # Syncs VERSION file to code
├── .github/
│   ├── CODEOWNERS            # Ownership mapping (enforced reviewers)
│   ├── dependabot.yml        # Dependency update automation
│   ├── ISSUE_TEMPLATE/       # Issue templates (bug/enhancement/support/design)
│   └── workflows/            # GitHub Actions CI workflows
│       ├── unit.yml          # Matrix RSpec unit tests (Windows + Ruby versions)
│       ├── lint.yml          # Lint (Cookstyle/Rubocop) pipeline
│       └── ci-main-pull-request-checks.yml # Centralized org pipeline stub
├── lib/                      # Library source code
│   └── mixlib/
│       ├── cli.rb            # Core CLI mixin implementation
│       └── cli/
│           ├── formatter.rb  # Option formatting utilities
│           └── version.rb    # VERSION constant (synced from VERSION file)
├── spec/                     # RSpec test suite
│   ├── spec_helper.rb        # RSpec configuration
│   └── mixlib/
│       ├── cli_spec.rb       # Tests for Mixlib::CLI behaviors
│       └── cli/
│           └── formatter_spec.rb # Tests for formatter helpers
└── .github/copilot-instructions.md # (This file)
```

Protected / DO NOT MODIFY without explicit approval: `LICENSE`, `NOTICE`, `CODE_OF_CONDUCT.md`, `CONTRIBUTING.md`, `.github/workflows/*` (unless task explicitly targets them), `.expeditor/config.yml`.

---
## Tooling & Ecosystem

| Aspect | Details |
|--------|---------|
| Primary Language | Ruby (>= 3.1) |
| Package Type | RubyGem (`mixlib-cli`) |
| Test Framework | RSpec (see `spec_helper.rb`) |
| Linting | Cookstyle (Rubocop variant) via `cookstyle --chefstyle -c .rubocop.yml` |
| Tasks | `rake spec`, `rake docs` |
| CI (GH Actions) | `unit.yml`, `lint.yml`, `ci-main-pull-request-checks.yml` |
| Release Automation | Expeditor (.expeditor) + Rubygems publish pipeline |
| Coverage Tooling | Not explicitly configured; recommend adding SimpleCov for coverage tracking if needed (see Testing & Coverage section). |
| Dependency Updates | Dependabot (`dependabot.yml`) |

Implicit Coverage Expectation: Maintain or raise overall effective test coverage above 80%. If formal measurement missing, implement SimpleCov in `spec/spec_helper.rb` (only if a task explicitly requires coverage instrumentation) and iterate until thresholds are met.

---
## MCP (Jira) Integration

All Jira interactions MUST use the `atlassian-mcp-server` MCP endpoint (not ad-hoc HTTP or manual copy). When a Jira ID is provided (e.g., `ABC-123`):

1. Acquire Issue:
   - Action (conceptual invocation): `atlassian-mcp-server.getJiraIssue(issueIdOrKey=ABC-123)`
2. Parse Fields:
   - Summary, Description
   - Acceptance Criteria (parse bullet lists / Gherkin blocks)
   - Story Points (if present)
   - Linked Issues (dependencies, blockers)
3. Produce a Structured Plan:
   - Design Overview
   - Impacted Files / Classes
   - Data / API considerations
   - Test Strategy (unit + edge cases)
   - Risk & Mitigations
4. Present plan to user with confirmation gate.
5. Proceed ONLY after explicit “yes”.

Example Interaction Pattern:
```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chef/mixlib-cli](https://github.com/chef/mixlib-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
