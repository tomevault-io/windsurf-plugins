---
trigger: always_on
description: start: "🎨 Starting design phase..."
---

version: "1.0"

# Communication settings
communication:
  emojis:
    design:
      start: "🎨 Starting design phase..."
      thinking: "🤔 Analyzing design requirements..."
      diagram: "📊 Creating system diagram..."
      complete: "✨ Design spec created in docs/design/"
    
    implement:
      start: "🚀 Starting implementation..."
      progress: "⚙️ Implementing features..."
      testing: "🧪 Testing implementation..."
      complete: "✅ Implementation finished"
    
    validate:
      start: "🔍 Starting validation..."
      checking: "📋 Checking against facts..."
      issue: "⚠️ Found compliance issue..."
      pass: "✅ Validation passed"
      fail: "❌ Validation failed"
      complete: "📝 Validation report created in docs/validation/"
    
    revise:
      start: "🔄 Starting revision..."
      progress: "📝 Making changes..."
      complete: "✨ Revision complete"
    
    refactor:
      start: "🔧 Starting refactoring..."
      progress: "🔨 Refactoring in progress..."
      metrics: "📊 Checking quality metrics..."
      complete: "✨ Refactoring complete"

# Documentation settings
documentation:
  output_dir: "./docs"
  templates:
    design:
      path: "docs/design"
      files:
        - name: "{timestamp}-design-spec.md"
          sections:
            - "Overview"
            - "Architecture"
            - "Components"
            - "Data Flow"
            - "Technical Decisions"
    
    validate:
      path: "docs/validation"
      files:
        - name: "{timestamp}-validation-report.md"
          sections:
            - "Executive Summary"
            - "Validation Context"
            - "Required Facts"
              subsections:
                - "Compliant"
                - "Non-compliant"
                - "Not Applicable"
            - "Recommended Facts"
              subsections:
                - "Implemented"
                - "Missing"
            - "Performance Metrics"
            - "Security Analysis"
            - "Code Quality Metrics"
            - "Issues"
              subsections:
                - "Critical"
                - "Major"
                - "Minor"
            - "Recommendations"
            - "Next Steps"

# Operation modes configuration
modes:
  design:
    description: "Architecture and system design phase"
    allowed_operations:
      - read_file
      - list_files
      - search_files
      - list_code_definition_names
      - write_to_file:
        extensions: [".md", ".mdx", ".mmd"]
      - access_mcp_resource
      - use_mcp_tool
      - ask_followup_question
    restrictions:
      - "No code modifications allowed"
      - "Can only create/modify markdown and mermaid files"
      - "Focus on architecture documentation and diagrams"

  implement:
    description: "Code implementation phase"
    allowed_operations:
      - all
    restrictions:
      - "Must follow design documents"
      - "Must validate against facts"

  validate:
    description: "Verification and compliance checking"
    allowed_operations:
      - read_file
      - list_files
      - search_files
      - list_code_definition_names
      - write_to_file:
        extensions: [".md"]
      - browser_action
      - execute_command:
        type: "read-only"
      - access_mcp_resource
      - use_mcp_tool
      - ask_followup_question
    restrictions:
      - "No code modifications allowed"
      - "Can only create validation reports in markdown"
      - "Can run tests and analysis tools"

  revise:
    description: "Code modification and improvement"
    allowed_operations:
      - all
    restrictions:
      - "Must maintain existing architecture"
      - "Changes must be validated against facts"

  refactor:
    description: "Code restructuring and optimization"
    allowed_operations:
      - all
    restrictions:
      - "Must maintain existing functionality"
      - "Must improve code quality metrics"
      - "Changes must be validated against facts"

# Facts integration settings
facts:
  # Automatically save decisions made during development
  auto_save: true
  
  # Default settings for new facts
  defaults:
    strictness: "RECOMMENDED"  # Default strictness level
    min_version: "1.0.0"      # Minimum version compatibility
    max_version: "*"          # Maximum version compatibility ("*" means latest)
    
  # Validation settings
  validation:
    # Validate all proposals against existing facts
    validate_proposals: true
    # Stop if validation fails for REQUIRED facts
    strict_validation: true
    # Categories to validate against
    categories: [
      "ARCHITECTURE_PATTERN",
      "DATABASE",
      "FRONTEND",
      "BACKEND",
      "DOCUMENTATION"
    ]
    
    # Validation stages
    stages:
      pre_planning:
        enabled: true
        categories:
          - ARCHITECTURE_PATTERN
          - PROJECT_STRUCTURE
          - FRAMEWORK
          - DATABASE
          - SECURITY
        actions:
          on_validation:
            - check_required_facts
            - determine_tools
            - set_constraints
          on_failure:
            - block_planning
            - generate_requirements
      
      post_planning:
        enabled: true
        categories:
          - ARCHITECTURE_PATTERN
          - DESIGN_PATTERN
          - PROJECT_STRUCTURE

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [andrewhopper/hivemind](https://github.com/andrewhopper/hivemind) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
