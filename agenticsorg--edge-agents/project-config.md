---
trigger: always_on
description: required: true                         # Ensure all code changes are backed by tests
---

# SPARC Focus: Specification - Define project objectives, requirements, and user scenarios
test_coverage:
  required: true                         # Ensure all code changes are backed by tests
  minimum_percentage: 80                 # Set a baseline of 80% test coverage
  exclude_paths:                         # Exclude paths that do not require testing
    - test/fixtures
    - scripts

# SPARC Focus: Specification - Set guidelines for code quality
lint_rules:
  disable_requires_approval: true        # Require explicit approval to disable lint rules
  enforce_eslint: true                   # Enforce ESLint for code consistency
  enforce_prettier: true                 # Enforce Prettier for consistent formatting

# SPARC Focus: Specification - Define styling and UI/UX guidelines
styling:
  use_tailwind: true                     # Utilize Tailwind CSS for standardized design
  prefer_functional_components: true     # Prefer functional components for UI development
  css_variables_location: webview-ui/src/index.css  # Central location for CSS variable definitions

# SPARC Focus: Specification - Environment-specific configurations
environments:
  python:
    virtual_env: true                    # Use a virtual environment for Python projects
    linter: flake8                       # Lint Python code with Flake8
    formatter: black                     # Format Python code with Black
  node:
    package_manager: npm                 # Use npm as the package manager
    test_runner: jest                    # Use Jest for running JavaScript tests
  deno:
    enabled: true                        # Enable Deno support
    version: ">=1.30.0"                  # Minimum Deno version required

# SPARC Focus: Pseudocode & Architecture - Model configuration for AI integration
provider:
  default: google/gemini-2.5-pro-experimental   # Default AI model provider
thinkingProvider:
  model: google/gemini-2.0-flash                # Model for quick, responsive analysis
docModel: google/gemini-2.0-pro                 # Model for processing documentation

# SPARC Focus: Pseudocode - Operational mode for code processing
mode: code                                      # Set the mode to 'code' for development tasks

# SPARC Focus: Architecture - Real-time updates configuration
real_time_updates:
  enabled: true                                 # Enable real-time context updates
  update_triggers:
    project_related:
      - documentation_gap                       # Trigger when documentation is incomplete
      - knowledge_update                        # Trigger upon new information or updates
    system_related:
      - error_pattern                           # Trigger when repeated error patterns are found
      - performance_insight                     # Trigger when performance insights become available

# SPARC Focus: Refinement - Use unified diffs for structured code edits
edit_format: unified_diff                       # Adopt unified diff for clear code modifications
high_level_edits: true                          # Encourage holistic, high-level changes
exclude_line_numbers: true                      # Exclude line numbers for cleaner diffs

# SPARC Focus: Architecture - Mode switching for multiple development contexts
mode_switching:
  enabled: true                                 # Allow automatic mode switching
  preserve_context: true                        # Retain context during mode transitions

# SPARC Focus: Pseudocode - Intent-based triggers for switching modes
intent_triggers:
  code:
    - implement                                 # Switch to code mode for implementing features
    - create                                    # Switch to code mode for creating components
    - build                                     # Switch to code mode for building modules
    - fix                                       # Switch to code mode for fixing issues
  architect:
    - design                                    # Switch to architect mode for designing systems
    - structure                                 # Switch to architect mode for structural planning
    - plan                                      # Switch to architect mode for project planning

# SPARC Focus: Architecture - File-based triggers for dynamic mode switching
file_triggers:
  - pattern: "\.tsx$"                           # Activate code mode for TSX files
    target_mode: code
    condition: file_edit
  - pattern: "\.md$"                            # Activate document mode for Markdown files
    target_mode: document
    condition: file_create

# SPARC Focus: Refinement - Terminal command management
terminal:
  allowed_commands:
    - npm test                                  # Run JavaScript tests
    - npm install                               # Install Node dependencies
    - tsc                                       # Compile TypeScript code
    - git log                                   # Show commit history
    - git show                                  # Display Git object content
    - cd                                        # Change directory
    - pip                                       # Install Python packages
    - docker                                    # Execute Docker commands

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agenticsorg/edge-agents](https://github.com/agenticsorg/edge-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
