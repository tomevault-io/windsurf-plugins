---
trigger: always_on
description: Guidelines for using and extending the template system in Atuin runbooks.
---

# Atuin Runbook Template System

Guidelines for using and extending the template system in Atuin runbooks.

<rule>
name: atuin_template_system
description: Standards for working with the template system in Atuin runbooks

filters:
  - type: file_path
    pattern: "src/components/runbooks/editor/blocks/.*"
  - type: file_path
    pattern: "backend/src/templates\\.rs"
  - type: file_path
    pattern: "src/state/templates\\.ts"

actions:
  - type: suggest
    message: |
      # Atuin Runbook Template System
      
      Atuin runbooks have a powerful template system that allows for dynamic content generation and variable substitution.
      
      ## Overview
      
      The template system uses [MiniJinja](mdc:https:/docs.rs/minijinja/latest/minijinja) (Rust implementation of Jinja2) and provides:
      
      - Access to document structure and blocks
      - Variable substitution
      - Template expressions
      
      ## Template Variables
      
      There are three main types of variables:
      
      1. **Environment Variables** (`Env` blocks) - Used for shell environment
      2. **Template Variables** (`Var` blocks) - Used for template substitution, synced across users
      3. **Local Variables** (`LocalVar` blocks) - Used for private variables (e.g., credentials), not synced across users
      
      ## Template State Structure
      
      The template state consists of:
      
      ```rust
      struct TemplateState {
          doc: Option<DocumentTemplateState>,  // Document structure info
          var: HashMap<String, Value>,         // Template variables
      }
      ```
      
      Where `DocumentTemplateState` contains:
      
      ```rust
      struct DocumentTemplateState {
          first: BlockState,                   // First block in the document
          last: BlockState,                    // Last block in the document
          content: Vec<BlockState>,            // All blocks in the document
          named: HashMap<String, BlockState>,  // Blocks with names
          previous: BlockState,                // Block before the current one
      }
      ```
      
      ## Creating Template Variables
      
      ### Shared Variables (Synced)
      
      Variables can be created using the `Var` block:
      
      1. Insert a Var block using the slash menu (/Template Variable)
      2. Set a name and value
      3. The variable will be stored in both the block props AND the backend state
      4. These variables are synced across all users
      
      ### Private Variables (Not Synced)
      
      For sensitive values like credentials, use the `LocalVar` block:
      
      1. Insert a LocalVar block using the slash menu (/Local Variable)
      2. Set a name (synced) and value (private)
      3. Only the variable name is stored in block props and synced
      4. The value is stored only in the backend state for the current user
      5. Other users see the name but not the value
      
      ## Using Template Variables
      
      Both shared and private variables can be referenced using the `{{ var.name }}` syntax:
      
      ```
      {{ var.username }}
      {{ var.password }}  <!-- If password is a LocalVar, each user's own value will be used -->
      ```
      
      ## Implementation Details
      
      ### Frontend
      
      #### Shared Variables (Var)
      
      - Both name and value are stored in UI component props (synced)
      - Changes are synced to the backend via the `set_template_var` command
      
      #### Private Variables (LocalVar)
      
      - Only the name is stored in UI component props (synced)
      - The value is stored only in backend state via `set_template_var` command
      - Values are retrieved via `get_template_var` command
      
      ### Backend
      
      - All template variables are stored in `AtuinState.runbook_output_variables`:
      
      ```rust
      // Map of runbook -> variable name -> variable value
      pub runbook_output_variables: Arc<RwLock<HashMap<String, HashMap<String, String>>>>
      ```
      
      - Template processing happens in the `template_str` command
      - Variables are passed to the template engine when rendering
      
      ## Command Interface
      
      ```rust
      #[tauri::command]
      pub async fn set_template_var(
          state: tauri::State<'_, crate::state::AtuinState>,
          runbook: String,
          name: String,
          value: String,
      ) -> Result<(), String> {
          state
              .runbook_output_variables
              .write()
              .await
              .entry(runbook)
              .or_insert(HashMap::new())
              .insert(name, value);
      
          Ok(())
      }
      
      #[tauri::command]
      pub async fn get_template_var(
          state: tauri::State<'_, crate::state::AtuinState>,
          runbook: String,
          name: String,
      ) -> Result<Option<String>, String> {
          let value = state
              .runbook_output_variables
              .read()
              .await
              .get(&runbook)
              .and_then(|vars| vars.get(&name))
              .cloned();
      
          Ok(value)
      }
      ```
      
      ## Template Processing Flow
      

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [atuinsh/desktop](https://github.com/atuinsh/desktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
