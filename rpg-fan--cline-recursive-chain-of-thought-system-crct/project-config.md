---
trigger: always_on
description: This outlines the fundamental principles, required files, workflow structure, and essential procedures that govern CRCT, the overarching framework within which all phases of operation function. Specific instructions and detailed procedures are provided in phase-specific plugin files in `.clinerules/`.
---

# Welcome to the Cline Recursive Chain-of-Thought System (CRCT)

This outlines the fundamental principles, required files, workflow structure, and essential procedures that govern CRCT, the overarching framework within which all phases of operation function. Specific instructions and detailed procedures are provided in phase-specific plugin files in `.clinerules/`.

**Important Clarifications:** The CRCT system operates in distinct *phases* (Set-up/Maintenance, Strategy, Execution, Cleanup/Consolidation), controlled **exclusively** by the `next_phase` setting in `.clinerules`. "Plan Mode" or any other "Mode" is independent of this system's *phases*. Plugin loading is *always* dictated by `next_phase`.

The dependencies in tracker grids (e.g., `pso4p`) are listed in a *compressed* format. **Do not attempt to decode dependency relations manually**, this is what commands like `show-dependencies` and `show-placeholders` are for.
*Do not rely on what you assume are 'p' relations in the raw grid output. The output of `show-dependencies` is the *only* valid source for viewing dependency relationships.*
**Example**: `python -m cline_utils.dependency_system.dependency_processor show-dependencies --key 3Ba2`
* If "3Ba2" is globally unique, this works directly.
* If "3Ba2" is globally ambiguous (e.g., multiple files/items share the base key "3Ba2"), the system will list all global instances like `3Ba2#1 (path/to/A)`, `3Ba2#2 (path/to/B)`, and prompt you to re-run the command with the specific instance, e.g., `show-dependencies --key 3Ba2#1`.

*`python -m cline_utils.dependency_system.dependency_processor` is a CLI operation and should be used with the `execute_command` tool.*

## Mandatory Initialization Procedure

**At initialization the LLM MUST perform the following steps, IN THIS ORDER:**
    1. **Read `.clinerules/default-rules.md`**: Determine `current_phase`, `last_action`, and `next_phase`. Note: `.clinerules` is now a directory; the authoritative rules live in `.clinerules/default-rules.md`. Legacy fallbacks may exist, but all tooling should prefer `default-rules.md`.
    *Note: the `next_action` field may not be relevant if you have just been initialized, defer to `activeContext.md` to determine your next steps. If you see references to "MUP" in any context related to your next actions/steps in `.clinerules` or `activeContext.md` ignore that action/step-it is a relic left over from the last session and not your concern.*
    2. **Load Plugin**: Based on `next_phase` indicated in `.clinerules/default-rules.md`, load the corresponding plugin from `.clinerules/`. **YOU MUST LOAD THE PLUGIN INSTRUCTIONS. DO NOT PROCEED WITHOUT DOING SO.**
    3. **Read Core Files**: Read the specific files listed in Section II below. Do not re-read these if already loaded in the current session.
    4. **Determine the virtual environment**: Search the project root for common virtual environment names (venv, .venv, etc.), if needed ask the user for the correct path to the environment.
    5. **Activate Environment**: Ensure the virtual environment is active before executing commands (Windows: `.\.venv\Scripts\Activate.ps1`, then run as `.\.venv\Scripts\python.exe -m ...`). Create if one does not exist.
      - May not follow exact ".venv" naming convention, check to see if a venv exists in the root directory.
    **FAILURE TO COMPLETE THESE INITIALIZATION STEPS WILL RESULT IN ERRORS AND INVALID SYSTEM BEHAVIOR.**

## Guidelines for File Modification Tool Usage

When modifying files, selecting the most appropriate and token-efficient tool is crucial for system performance and operational cost-effectiveness. Adhere to the following prioritization and guidelines:

1. **`insert_content`**:
    * **Use Case**: This tool should be your **primary choice** when the task involves **only adding new content** to a file without altering or deleting any existing lines.
    * **Examples**:
        * Appending new entries, such as version updates or feature additions, to a `changelog.md` file.
        * Inserting a new function, class definition, or a block of import statements into a pre-existing code file at a specific, clearly defined location.
        * Adding new configuration items to a list or a new key-value pair to a dictionary/object within a configuration file (e.g., JSON, YAML) where the insertion point is precise.
    * **Rationale**: `insert_content` is highly efficient as it only requires transmitting the content to be inserted and the target line number, minimizing token usage compared to rewriting larger portions of the file.
    *Be very careful to match the **indention** of the content you are inserting to*

2. **`search_and_replace` or `apply_diff`**:
    * **Use Case**: Utilize these tools when you need to **edit, modify, or change existing content within localized areas** of a file.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RPG-fan/Cline-Recursive-Chain-of-Thought-System-CRCT-](https://github.com/RPG-fan/Cline-Recursive-Chain-of-Thought-System-CRCT-) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
