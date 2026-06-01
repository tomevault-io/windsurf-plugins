---
trigger: always_on
description: When new code, features, or experiments are added, it's crucial to update the relevant documentation to ensure discoverability and maintainability.
---

---

## 7. Documentation Workflows

### Updating Documentation for New Features/Experiments

When new code, features, or experiments are added, it's crucial to update the relevant documentation to ensure discoverability and maintainability.

1.  **Identify Relevant Docs:** Determine which documentation files need updating (e.g., main `README.md`, `experiments/README.md`, `developers_guide.md`).
2.  **Analyze Existing Conventions:** Read the target documentation to understand its structure, tone, and formatting conventions for similar items.
3.  **Synthesize New Content:** Extract key information from the new feature's source code or its own README to create a concise and accurate description.
4.  **Propose Changes:** Present the proposed documentation changes to the user for review and approval before applying them. Use markdown blocks to clearly show the additions or modifications.
5.  **Apply Changes:** Use the `replace` or `write_file` tool to apply the approved changes.

### Integrating External Agent Skills

When importing, copying, or adding a new Agent Skill to the repository:

1.  **Verify Structure**: Ensure the skill folder has a conforming `SKILL.md` file with YAML frontmatter at its root.
2.  **Metadata Extraction**: Read the `SKILL.md` file to extract the `name`, `description`, and other relevant attributes from its YAML frontmatter.
3.  **Synchronize READMEs**: Dynamically and consistently update both the main workspace `README.md` (or the component-level README) and the skills directory `README.md` with the extracted skill name and description to ensure discoverability.
4.  **Issue Tracking**: Log the skill addition in the beads (`bd`) issue tracker by creating, claiming, and closing a dedicated issue for the work.


## 8. Mesop Development Practices

### Refactoring and State Management
*   **Dynamic UI Binding:** UI components must never hardcode lists of available models for specific modes. Instead, add helper functions to the respective `config/*_models.py` file (e.g., `get_models_by_mode`) and dynamically generate UI dropdowns based on the model configuration's declared capabilities.
*   **Component Modularity:** Mesop pages can quickly grow to thousands of lines. Proactively refactor UI sections (e.g., `upload_ui`, `controls`, `gallery`) into separate files within a `components/` directory to keep page files tractable.
*   **Event Handler Factories:** When sharing UI components across multiple pages that maintain their own `PageState` classes, use closure factories (e.g., `def get_on_click_handler(state_class): ...`) to generate reusable event handlers. This avoids duplicating state-mutation logic across pages.
*   **Safe Python Refactoring:** When executing large refactors via CLI tools, be extremely cautious with string replacements (`sed` or python scripts) on nested Mesop structures (`with me.box():`). Missing or extra spaces will cause fatal `IndentationError`s. Prefer targeted replacements or abstracting the block into a function first. When using python scripts to modify large configuration lists or Mesop component trees, avoid greedy regex replacements (`re.sub`). Rely on exact string matching or AST manipulation to prevent accidentally corrupting adjacent, structurally identical blocks.

### Component Styling Gotchas
*   **Button Types:** The `me.content_button` component strictly enforces the `type` argument as a literal: `'raised'`, `'flat'`, `'stroked'`, or `'icon'`. Using invalid MD3 concepts like `'tonal'` will cause a Pydantic `ValidationError` and crash the UI block.
*   **Interactive Toggles:** When changing the background of a button dynamically (e.g., to indicate selection), ensure you also dynamically update the `color` property of its children (e.g., `me.theme_var("on-primary") if is_selected else me.theme_var("on-surface")`) so icons and text don't become invisible.
*   **Consistent Workflow Page Layouts:** When designing multi-step generative workflows (like Storyboarder, Veo, or Interior Design), prefer a balanced two-column top row: placing the main story concept or text prompter in a wider left column (e.g., 60% width) and optional reference media uploaders/character setups in a narrower right column (e.g., 40% width). Nested configuration/settings selectors should reside below them inside a collapsible panel.
*   **Divider Styling Constraint:** The `me.divider` component strictly rejects the `style` argument. To apply margins, padding, or custom alignment around a line divider, wrap `me.divider()` inside a styled `me.box`.
*   **Auto-dismissing Snackbars:** Do not leave `state.show_snackbar = True` active indefinitely. Always implement a generator helper function to auto-dismiss snackbars:
    ```python
    def show_snackbar(state: PageState, message: str):
        state.snackbar_message = message
        state.show_snackbar = True
        yield
        time.sleep(3)
        state.show_snackbar = False
        yield
    ```

### SDK Integration Nuances

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GoogleCloudPlatform/vertex-ai-creative-studio](https://github.com/GoogleCloudPlatform/vertex-ai-creative-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
