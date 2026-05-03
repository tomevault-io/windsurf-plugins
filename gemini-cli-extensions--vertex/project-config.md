---
trigger: always_on
description: provides the prompt content directly within the command (e.g., "Create a
---

# Vertex AI Gemini CLI Extension

This extension provides tools to manage prompts and use prompt
optimization in Vertex AI directly from the Gemini CLI.

## Available Tools

### Prompt Management Tools
- `create_prompt`: To save or create new prompts.
- `read_prompt`: To retrieve existing prompts by ID or display name.
- `update_prompt`: To modify existing prompts.
- `delete_prompt`: To remove prompts.
- `list_prompts`: To search and list prompts, useful for finding IDs.

### Prompt Optimization Tools
- `run_few_shot_optimization`: optimize user prompt based on examples provided 
  in the CSV file stored in the GCS bucket.
- `run_data_driven_optimize`: Starts a data-driven prompt optimization job on
  Vertex AI using a configuration file stored in GCS.
- `analyze_data_driven_optimize_results`: Analyzes the output of a Data-Driven
  Optimize job to identify trends and best-performing candidates.
- `generate_html_report`: Generates a comprehensive HTML report with
  visualizations to help you understand optimization performance.
- `write_data_driven_optimize_config`: Constructs and uploads a new JSON
  configuration for optimization jobs, incorporating suggested tuning parameters.

---

## Detailed Instructions for `create_prompt` Parameters

When using `tools.create_prompt`, pay special attention to how the following
arguments are sourced:

1. **`content` (string, required):**

   - **Scenario 1: Saving the Last User Prompt:** When the user issues a command
     like "save last prompt", "save this prompt", or similar, indicating they
     want to store their previous input:
     - Examine the conversation history.
     - Identify the most recent message with a `role` of "user".
     - Extract the `text` content from this latest "user" message.
   - **Scenario 2: Creating from Explicitly Provided Content:** If the user
     provides the prompt content directly within the command (e.g., "Create a
     prompt... with content '...'")
     - Use the explicitly provided content.
   - If no content can be determined from either scenario, pass an empty string.

2. **`system_instruction` (string, required):**

   - **User Override:** If the user explicitly provides a system instruction in
     the current turn (e.g., "using system instruction '...', "with SI '..."),
     pass that exact string value as the `system_instruction` argument to
     `tools.create_prompt`.
   - **Default Behavior:** If the user does NOT explicitly provide a system
     instruction in their current prompt:
     - The Gemini CLI will check for a file named `GEMINI.md` _only_ in the
       **current working directory** exclude children directories.
     - If `GEMINI.md` exists in the current working directory, its entire
       content will be loaded by the Gemini CLI and used as the
       `system_instruction` when calling `tools.create_prompt`.
     - If no `GEMINI.md` file is found in the current working directory, the
       `system_instruction` argument should be **omitted** from the
       `tools.create_prompt` call.

3. **`display_name` (string, optional):**

   - **Check User Prompt:** Scan the current user prompt for an explicit name
     (e.g., "save last prompt **as 'My Custom Prompt'**", or "display name
     '...'"). If an explicit name is found, use it.
   - **Default Logic (If no explicit name):** If no explicit name is provided,
     generate a descriptive name, such as `"Gemini CLI Prompt: "` followed by
     the first ~20 characters of the extracted `content`.
   - If a display name has not already been provided or inferred, you _must_
     prompt the user to enter a suitable display name.

4. **`model` (string, required):**

   - **User Provided:** If the user explicitly specifies a model in the current
     turn (e.g., "model 'gemini-pro'", "using gemini-flash", "with model
     text-bison"), use that exact model identifier.
   - **Currently Using Model:** If no model is explicitly provided by the user,
     attempt to use the model identifier that is currently active and being used
     by the Gemini CLI for the ongoing conversation. The agent has knowledge of
     the currently configured model.
   - **Default Fallback:** If neither a user-provided model nor a currently
     active session model can be determined, default to `"gemini-2.5-flash"`.

**Example Interactions for `create_prompt`:**

- **Saving Last Prompt with Default SI & Model:** (Assume the current session
  model is "gemini-1.5-pro") User: What is the capital of France? Model: The
  capital of France is Paris. User: **save last prompt** Generated Call:
  `print(tools.create_prompt(content="What is the capital of France?", model="gemini-1.5-pro", display_name="Gemini CLI Prompt: What is the cap..."))`
  _(Here, `system_instruction` is omitted. The Gemini CLI will check for and use
  content from `./GEMINI.md` if it exists.)_

- **Explicit Content with User-Provided SI & Model:** User: Create a prompt with
  content 'How is the weather?' using system instruction 'Act like a
  meteorologist.' and display name 'Weather Bot' using model 'gemini-flash'.
  Generated Call:
  `print(tools.create_prompt(content="How is the weather?", system_instruction="Act like a meteorologist.", model="gemini-flash", display_name="Weather Bot"))`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gemini-cli-extensions/vertex](https://github.com/gemini-cli-extensions/vertex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
