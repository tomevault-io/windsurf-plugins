---
trigger: always_on
description: Comprehensive specification management system for creating, validating, and tracking requirements
---

# Specification Management System

Rule for creating, validating, and managing specifications throughout the development lifecycle.

<rule>
name: specification_management
filters:
  # Request specs filters
  - type: event
    pattern: "user_request"
  - type: command
    pattern: "spec"
  - type: command
    pattern: "requirement"
  
  # Validation filters
  - type: event
    pattern: "spec_create"
  - type: event
    pattern: "spec_update"
  - type: file_change
    pattern: ".cursor/specs/*.md"
  - type: command
    pattern: "validate"

actions:
  #
  # SECTION 1: SPECIFICATION CREATION
  #
  
  - type: react
    event: "user_request"
    conditions:
      - pattern: "implement|create|build|develop"
    action: |
      # Create specifications before implementation
      
      When you ask me to implement something, I'll:
      
      1. Check if specifications exist for the requested feature
      2. If specifications don't exist:
         - Create a new specification file in `.cursor/specs/`
         - Organize specs by domain/component
         - Define clear requirements as checkable items
         - Add acceptance criteria
         - Link to related specifications if applicable
      3. Update the specifications index `SPECS.md`
      4. Trigger task creation (via development_workflow_system) after specs are created
      
      This ensures all implemented features have proper specifications 
      before development begins.

  - type: react
    conditions:
      - pattern: "spec create|create spec"
    action: |
      # Explicitly create a new specification
      
      I'll create a new specification file:
      
      1. Identify the appropriate domain for the specification
      2. Create the spec file in `.cursor/specs/[domain]/[name].md`
      3. Structure the spec with:
         - Clear title and description
         - Numbered requirements as checkable items
         - Acceptance criteria
         - Notes or context
      4. Update the specifications index `SPECS.md`
      5. Capture any learnings about the specification process (via knowledge_management rule)
      
      This creates a formal specification that can be referenced during development.

  - type: react
    conditions:
      - pattern: "spec update|update spec"
    action: |
      # Update an existing specification
      
      I'll update the specified specification:
      
      1. Find the existing specification file
      2. Add or modify requirements as needed
      3. Update the completion status of requirements
      4. Refresh any related information
      5. Update the specifications index if necessary
      6. Notify the development workflow system of changes (via development_workflow_system)
      
      This ensures specifications stay current as requirements evolve.
  
  #
  # SECTION 2: SPECIFICATION VALIDATION
  #
  
  - type: react
    conditions:
      - pattern: "spec validate|validate spec"
    action: |
      # Validate a specification against quality standards and implementation
      
      I'll perform a comprehensive validation of the specification:
      
      1. Analyze specification quality:
         - Check for vague terms (should, would, could, etc.)
         - Identify non-atomic requirements (multiple requirements in one)
         - Verify all required sections exist (description, requirements, acceptance criteria)
         - Assess overall specification clarity and completeness
      
      2. Validate implementation status:
         - Trace requirements to actual code
         - Check if completed requirements have corresponding implementations
         - Identify discrepancies between spec completion status and actual code
         - Calculate implementation coverage percentage
      
      3. Review test coverage:
         - Look for test files related to this specification
         - Verify test existence for key requirements
         - Identify gaps in test coverage
      
      4. Provide actionable recommendations:
         - Suggest quality improvements
         - Highlight missing implementations
         - Recommend test additions
      
      5. Capture validation learnings (via knowledge_management rule)
      
      The validation report will be saved to `.cursor/output/spec_validation_[filename]_[timestamp].md`

  - type: react
    conditions:
      - pattern: "spec format|format spec"
    action: |
      # Format a specification to improve its quality
      
      I'll improve the specification format:
      
      1. Ensure proper structure:
         - Add title if missing
         - Create standard sections (Description, Requirements, Acceptance Criteria, Notes)
         - Format requirements as proper checkboxes
      
      2. Improve requirement quality:
         - Split non-atomic requirements (containing "and")
         - Convert vague requirements to specific ones
         - Ensure consistent formatting
      
      3. Backup the original specification
      
      This improves specification quality while preserving all original content.

  - type: react
    conditions:
      - pattern: "spec completeness|completeness check"
    action: |
      # Check specification completeness across the project
      

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AndreRatzenberger/cursor-rules](https://github.com/AndreRatzenberger/cursor-rules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
