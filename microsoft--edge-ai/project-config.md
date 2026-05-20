---
trigger: always_on
description: Comprehensive coding guidelines and instructions for edge ai - Brought to you by microsoft/edge-ai
---


# General Instructions

Items in **HIGHEST PRIORITY** sections from attached instructions files override any conflicting guidance.

## **HIGHEST PRIORITY**

**Breaking changes:** Do not add backward-compatibility layers or legacy support unless explicitly requested. Breaking changes are acceptable.

**Artifacts:** Do not create or modify tests, scripts, or one-off markdown docs unless explicitly requested.

**Comment policy:** Never include thought processes, step-by-step reasoning, or narrative comments in code.

* Keep comments brief and factual; describe **behavior/intent, invariants, edge cases**.
* Remove or update comments that contradict the current behavior. Do not restate obvious functionality.
* Do NOT add temporal or plan-phase markers (e.g. "Phase 1 cleanup", "... after migration", dates, or task references) to code files. When editing or updating any code files, always remove or replace these types of comments.

**Conventions and Styling:** Always follow conventions and styling in this codebase FIRST for all changes, edits, updates, and new files.

* Conventions and styling are in instruction files and must be read in with the `read_file` tool if not already added as an `<attachment>`.

**Proactive fixes:** Always fix problems and errors you encounter, even if unrelated to the original request. Prefer root-cause, constructive fixes over symptom-only patches.

* Always correct conventions and styling and comments.

**Deleting files and folders:** Use `rm` with the run_in_terminal tool when needing to delete files or folders.

**Edit tools:** Never use `insert_edit_into_file` tool when other edit and file modification tools are available.

## Repository Configuration

* **Default branch**: `origin/dev` — use as the base for all new branches, comparisons, and PR targets unless explicitly overridden.

### CRITICAL - Required Prompts & Instruction Compliance

**Context-first:** Evaluate the current user prompt, any attachments, target folders, repo conventions, and files already read.

**Discover & match (do this BEFORE any edit):**

* Run `<search-for-prompts-files>` using the rules below (see table).
* For each matched prompts/instructions/copilot file:
  * If it is NOT already provided as a full, non-summarized `<attachment>` in this conversation and NOT already fetched via `read_file`, then read it now.
  * Use read_file to **page through the entire file**: read **2,000 lines per call**; make additional calls until EOF.
  * If the file references other prompts/instructions/copilot files, **recursively read those** to completion under the same paging rule.

**Apply instructions:** Treat the union of all matched files as **HIGHEST PRIORITY** for this task.

**Re-check cadence:** Re-run discovery and re-read all matched instruction files if missing **before each major editing phase**.

<!-- <search-for-prompts-files> -->
## Prompts Files Search Process

When working with specific types of files or contexts, you must:

1. Detect patterns and contexts that match the predefined rules
2. Search for and read the corresponding prompts files
3. Read a minimum of 2000 lines from these files before proceeding with any changes

### Matching Patterns and Files for Prompts

| Pattern/Context                | Required Prompts Files                                 |
|--------------------------------|--------------------------------------------------------|
| Any deployment-related context | `./.github/prompts/deploy.prompt.md`                   |
| Any getting started context    | `./.github/prompts/getting-started.prompt.md`          |
| Any terraform context          | `./.github/instructions/terraform.instructions.md`     |
| Any bicep context              | `./.github/instructions/bicep.instructions.md`         |
| Any shell or bash context      | `./.github/instructions/shell.instructions.md`         |
| Any bash in src context        | `./.github/instructions/bash.instructions.md`          |
| Any python context             | `./.github/instructions/python-script.instructions.md` |
| Any C# or csharp context       | `./.github/instructions/csharp.instructions.md`        |

<!-- </search-for-prompts-files> -->

<!-- <component-and-blueprint-structure> -->
## Component and Blueprint Structure Understanding

Components follow a decimal naming convention for deployment order and are organized into discrete, self-contained units with specific deployment patterns.
Blueprints orchestrate multiple components to create complete infrastructure solutions.

### Grouping Organization

Components are organized in deployment-ordered groupings:

* **Template**: `**/src/{000}-{grouping_name}/**`
* **Cloud Infrastructure**: `**/src/000-cloud/**` - Azure cloud resources (000-099 range)
* **Edge Infrastructure**: `**/src/100-edge/**` - Edge cluster and IoT operations (100-199 range)
* **Applications**: `**/src/500-***/**` - Application workloads (500-599 range)
* **Utilities**: `**/src/900-***/**` - Tools and utilities (900-999 range)

### Component Organization Structure Template

Each component follows this mandatory directory structure:

```text
{grouping}/{000}-{component_name}/
├── README.md                    # Component documentation and usage

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/edge-ai](https://github.com/microsoft/edge-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
