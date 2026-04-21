---
trigger: always_on
description: Comprehensive knowledge management system for capturing, organizing, and applying project knowledge
---

# Knowledge Management System

Rule for capturing, organizing, refining, and applying knowledge throughout the project lifecycle.

<rule>
name: knowledge_management
filters:
  # Information tracking filters
  - type: event
    pattern: "knowledge_capture"
  - type: event
    pattern: "file_create"
  - type: event
    pattern: "file_change"
  - type: command
    pattern: "learn"
  - type: command
    pattern: "document"
  - type: event
    pattern: "conversation_insight"
  - type: file_change
    pattern: ".cursor/docs/*"
  
  # Learning refinement filters
  - type: event
    pattern: "learning_create"
  - type: event
    pattern: "learning_update"
  - type: file_change
    pattern: ".cursor/learnings/*.md"
  - type: command
    pattern: "knowledge"
  
  # Actionable insights filters
  - type: command
    pattern: "insight"
  - type: event
    pattern: "implementation_complete"
  - type: event
    pattern: "task_complete"
  - type: event
    pattern: "code_review"
  - type: file_change
    pattern: "src/*"

actions:
  #
  # SECTION 1: KNOWLEDGE CAPTURE
  #
  
  - type: react
    conditions:
      - pattern: "learn add|learning create"
    action: |
      # Create a new learning entry
      
      I'll create a structured learning document with:
      - Unique ID (format: LEARN-YYYY-MM-DD-NN)
      - Title and short description
      - Detailed description of the learning (including context related to **SOLID, TDD, SRP, Calisthenics, resilience** if applicable)
      - Relevant code files (using `Sources/`, `Tests/` paths), tasks (managed by development_workflow_system), specifications (managed by specification_management), and documents
      - Date of capture
      - Keywords for easy retrieval
      
      The learning will be stored in `.cursor/learnings/` 
      and indexed in `.cursor/LEARNINGS.md`

  - type: react
    conditions:
      - pattern: "document add|document create|document detect"
    action: |
      # Register a document in the knowledge base
      
      I'll register the document by:
      - Storing a copy in `.cursor/docs/` if it's not already there
      - Creating a learning entry about this document
      - Adding the document to the documents index
      - Extracting key information from the document
      - Creating cross-references to related content including specifications (managed by specification_management)
      
      This ensures all project documentation is centrally tracked
      and discoverable.

  - type: react
    event: "conversation_insight"
    action: |
      # Automatically capture important insights from our conversations
      
      When I identify valuable insights in our conversation, I'll:
      - Create a learning entry with the insight
      - Generate a unique ID for the learning
      - Categorize and tag the insight appropriately
      - Add context and relevant connections to tasks (managed by development_workflow_system)
      - Index the insight for future reference
      
      This ensures important information isn't lost.

  - type: react
    event: "file_change"
    conditions:
      - pattern: ".cursor/docs/.*"
    action: |
      # Automatically process new documents added to docs directory
      
      When a new document is added to `.cursor/docs/`, I'll:
      - Extract key information from the document
      - Create a learning entry linked to this document
      - Update the documents index
      - Identify related content including specifications (managed by specification_management)
      - Generate appropriate metadata
      
      This ensures all documentation is properly integrated into
      the knowledge base.
  
  #
  # SECTION 2: KNOWLEDGE ORGANIZATION & REFINEMENT
  #
  
  - type: react
    conditions:
      - pattern: "learn categorize|learning categorize"
    action: |
      # Categorize and organize learnings
      
      I'll organize all learnings into meaningful categories:
      
      1. Analyze all learning content to identify topics and themes
      2. Categorize learnings into domains like:
         - Architecture, Performance, Security, DevOps
         - UX, API, Database, Testing
         - Frontend, Backend, Mobile, Tooling
         - Process, Bugs, Documentation
      3. Create category files in `.cursor/learning_categories/`
      4. Generate a categories index with links to all categorized learnings
      5. Identify uncategorized learnings for further review
      
      This makes knowledge more discoverable by organizing it into logical domains.

  - type: react
    conditions:
      - pattern: "learn refine:(.*)"
    action: |
      # Refine a specific learning to enhance its value
      
      I'll refine the specified learning by:
      
      1. Extracting the core learning ID from the command
      2. Creating an enhanced version with additional sections:
         - Keywords extracted from content
         - Key takeaways for quick reference
         - Potential applications of this knowledge
         - Related learnings on similar topics
         - Improved formatting and organization
      3. Preserving all original content while adding refinements
      4. Adding a "Last Refined" date
      

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/supermarios77) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
