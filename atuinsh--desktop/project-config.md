---
trigger: always_on
description: Writing a new block component for atuin runbooks
---

# Creating Components for Atuin Runbooks

Guidelines for creating new component blocks for the Atuin runbook editor.

<rule>
name: atuin_component_creation
description: Standards for creating and implementing component blocks in Atuin Desktop

filters:
  - type: event
    pattern: "file_create"
  - type: file_path
    pattern: "src/components/runbooks/editor/blocks/.*"

actions:
  - type: suggest
    message: |
      # Creating Components for Atuin Desktop
      
      ## Component Types
      
      There are two primary types of components:
      
      1. **Passive Blocks** - Provide context or configuration for other blocks (e.g., Directory, Env, Var, Host)
      2. **Executable Blocks** - Run commands or execute actions (e.g., Run, Script, HTTP)
      
      ## Component Structure
      
      ```
      src/components/runbooks/editor/blocks/[ComponentName]/
      ├── ComponentName.tsx  (or index.tsx)
      └── index.ts           (exports the component)
      ```
      
      ## Implementation Steps
      
      1. **Create Component Directory**
      2. **Create Main Component File**:
         - Use `createReactBlockSpec` to define the block
         - Implement a React component for the UI
      3. **Create Index File** for exports
      4. **Register in Schema** (`create_editor.ts`)
      5. **Add to Menu Items** (`Editor.tsx`) in the appropriate group
      6. **Create Backend Command** if the component needs to interact with the system:
         - Create in `backend/src/commands/[category].rs`
         - Add module to `backend/src/commands/mod.rs` 
         - Register command in main.rs (`tauri::generate_handler!`)
      
      ## Component Specification Example
      
      ```typescript
      export default createReactBlockSpec(
        {
          type: "your-component-type", 
          propSchema: {
            property: { default: "default-value" },
          },
          content: "none",
        },
        {
          render: ({ block, editor }) => {
            return <YourComponent {...props} />;
          },
        }
      );
      ```
      
      ## Suggestion Menu Item Example
      
      ```typescript
      export const insertYourComponent = (schema: any) => (editor: typeof schema.BlockNoteEditor) => ({
        title: "Component Name",
        subtext: "Description of component",
        onItemClick: () => {
          insertOrUpdateBlock(editor, {
            type: "your-component-type",
          });
        },
        icon: <SomeIcon size={18} />,
        group: "Appropriate Group", // Network, Execute, Database, etc.
      });
      ```
      
      ## For Passive Variable Blocks
      
      When creating variable blocks (like Env or template Var):
      
      1. Create a React component with input fields for name and value
      2. Use props to manage state and sync with editor
      3. For template variables, invoke a Tauri command to update backend state:
      
      ```typescript
      // In your component's render function:
      const onUpdate = (name: string, value: string): void => {
        // Update block props locally
        editor.updateBlock(block, {
          props: { ...block.props, name, value },
        });
        
        // Update backend state via command
        if (name && currentRunbookId) {
          invoke("set_template_var", {
            runbook: currentRunbookId,
            name,
            value,
          }).catch(console.error);
        }
      };
      ```
      
      ## Backend Command Structure
      
      For components that need backend state:
      
      ```rust
      // In backend/src/commands/template.rs
      #[tauri::command]
      pub async fn set_template_var(
          state: tauri::State<'_, crate::state::AtuinState>,
          runbook: String,
          name: String,
          value: String,
      ) -> Result<(), String> {
          // Update state logic here
          state
              .runbook_output_variables
              .write()
              .await
              .entry(runbook)
              .or_insert(HashMap::new())
              .insert(name, value);
      
          Ok(())
      }
      ```
      
      ## Architecture Guidelines
      
      - Commands using the Tauri API should live in `@commands` directory
      - Runtime functionality (without Tauri dependencies) lives in `@runtime`
      - Use the component's `onUpdate` method to trigger backend state changes
      - For passive blocks, use `findFirstParentOfType` and `findAllParentsOfType` to discover block values
      
      ## Best Practices
      
      - Follow existing patterns for similar block types
      - Maintain consistent styling with other blocks
      - Add tooltips to explain functionality
      - Group components properly in the menu
      - For passive blocks, use `findFirstParentOfType` to find parent blocks
      - Use `@heroui/react` components and `lucide-react` icons
      - When updating backend state, check that required values are present

examples:
  - input: |
      Creating a new Host block component
    output: |
      // Host component (src/components/runbooks/editor/blocks/Host/HostSelect.tsx)
      
      import { Button, Tooltip } from "@heroui/react";
      import { HomeIcon } from "lucide-react";

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [atuinsh/desktop](https://github.com/atuinsh/desktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
