---
trigger: always_on
description: Okay, here's an AGENTS.md file outlining coding guidelines for the 'awe' project. This document is intended to ensure consistency, readability, and maintainability across the codebase for both Unreal Engine and Max for Live components.
---

Okay, here's an AGENTS.md file outlining coding guidelines for the 'awe' project. This document is intended to ensure consistency, readability, and maintainability across the codebase for both Unreal Engine and Max for Live components.
awe: Ableton Unreal Engine Integration - Coding Guidelines (AGENTS.md)
1. Introduction
This document outlines the coding guidelines and best practices for all contributors to the 'awe' project. Adhering to these guidelines will help maintain code quality, improve collaboration, and ensure the long-term stability and maintainability of the integration between Ableton Live and Unreal Engine. All agents (developers, contributors) are expected to follow these guidelines.
2. General Principles
 * Readability: Write code that is easy to read and understand. Prioritize clarity over overly clever or obscure solutions.
 * Consistency: Strive for consistency in naming, formatting, and architectural patterns throughout the project.
 * Simplicity (KISS): Keep It Simple, Stupid. Avoid unnecessary complexity.
 * Don't Repeat Yourself (DRY): Avoid code duplication. Utilize functions, classes, and reusable components.
 * Commenting:
   * Comment code that is complex, non-obvious, or critical.
   * Explain why something is done, not just what is being done (if the what is clear from the code).
   * Keep comments up-to-date with code changes.
 * Modularity: Design components to be as self-contained and reusable as possible.
 * Performance: Be mindful of performance implications, especially for real-time operations. Profile and optimize critical code paths.
 * Error Handling: Implement robust error handling and provide clear feedback to users or logs when errors occur.
3. Unreal Engine Specific Guidelines
Follow Epic Games' official coding standards and guidelines where applicable.
3.1. C++
 * Naming Conventions:
   * Adhere to Epic Games' C++ Naming Conventions (e.g., UClassName, FStructName, EMyEnum, bBooleanVariable, FunctionName(), VariableName).
   * Use clear, descriptive names for classes, functions, variables, and files.
 * Formatting:
   * Follow Epic's C++ code formatting style (indentation, bracing, spacing).
   * Use PascalCase for class, struct, enum, and function names.
   * Use PascalCase (with a b prefix for booleans) for member variables.
   * Use camelCase for local variables and function parameters.
 * Headers (.h files):
   * Use #pragma once for include guards.
   * Minimize include dependencies. Use forward declarations where possible.
   * Organize includes: Engine-specific, Plugin-specific, Standard library.
 * Comments:
   * Use // for single-line comments and /*... */ for multi-line comments.
   * Use Doxygen-style comments for documenting classes, functions, and significant variables for API documentation generation.
 * Performance:
   * Avoid unnecessary operations in performance-critical loops (e.g., Tick functions).
   * Use appropriate data structures for the task.
   * Profile code using Unreal Insights to identify bottlenecks.
 * Memory Management:
   * Understand and correctly use Unreal Engine's UObject memory management and garbage collection.
   * Use smart pointers (TSharedPtr, TUniquePtr, TWeakPtr) for non-UObject memory management where appropriate.
 * Logging:
   * Use UE_LOG for logging messages. Define appropriate log categories for the 'awe' plugin.
   * Use different verbosity levels (Error, Warning, Log, Verbose) appropriately.
3.2. Blueprints
 * Naming Conventions:
   * Prefix Blueprint assets appropriately (e.g., BP_MyActor, WBP_MyWidget, EUW_MyEditorTool, AS_MySound, SK_MySkeleton).
   * Variables: PascalCase (e.g., MyVariable, bIsActive).
   * Functions/Events: PascalCase (e.g., HandleInput, OnDataReceived).
 * Graph Organization:
   * Keep graphs clean, organized, and easy to follow.
   * Use comments (C key) to explain sections of logic.
   * Align nodes and wires neatly. Use reroute nodes to improve readability.
   * Break down complex logic into smaller functions or macros.
   * Group related nodes using comment boxes.
 * Variables:
   * Use appropriate data types.
   * Provide clear tooltips for public variables.
   * Organize variables into categories in the "My Blueprint" panel.
 * Performance:
   * Be mindful of operations performed on Event Tick. Minimize logic in Tick or use timers/event-driven approaches where possible.
   * Avoid unnecessary casting.
   * Understand the cost of Blueprint function calls, especially across different Blueprints.
 * Compilation:
   * Always compile Blueprints after making changes.
   * Address all compiler warnings and errors.
3.3. UMG (Unreal Motion Graphics) & Editor Utility Widgets (EUW) [1, 2, 3, 4, 5, 6, 7, 8]
 * Naming Conventions:
   * Widgets in the hierarchy should have clear, descriptive names (e.g., Button_Submit, Text_StatusMessage, ListView_ParameterMappings).
 * Hierarchy:
   * Maintain a clean and logical widget hierarchy in the UMG Designer.
 * Styling & Consistency:
   * Strive for visual consistency with the Unreal Editor's UI, especially for Editor Utility Widgets.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dfischer/awe](https://github.com/dfischer/awe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
