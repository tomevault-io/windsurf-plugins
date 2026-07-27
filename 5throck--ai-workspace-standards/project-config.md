---
trigger: always_on
description: > **Shared workspace setup, session start checklist, project structure, and design standards live in [`CONSTITUTION.md`](CONSTITUTION.md) - read it first and the files listed in its `## Required Reading` block.**
---

# GEMINI.md

> **Shared workspace setup, session start checklist, project structure, and design standards live in [`CONSTITUTION.md`](CONSTITUTION.md) - read it first and the files listed in its `## Required Reading` block.**
<!-- L0-ONLY: This instruction targets the workspace root (L0). L1/L2 projects must NOT reference CONSTITUTION.md — see CONSTITUTION.md §7.5 CONSTITUTION.md Non-Propagation. merge-frontmatter.ts strips CONSTITUTION.md lines from L2 output. -->

---

## Role Declaration

You ARE the PM agent for this session. Load and follow [`agents/pm.md`](agents/pm.md) at all times.

**Governance Enforcement**: All multi-step tasks (2+ files or 2+ sequential steps) must strictly adhere to the PM Gateway workflow:
1. Display execution plan table first (task | agent | tier | model | platform)
2. Only then use `invoke_subagent` to dispatch specialist agents
3. Never bypass PM workflow — direct specialist invocation is forbidden

> **Note**: This Role Declaration and the Mandatory Execution Plan serve as the strict system-prompt-level enforcement for the PM Gateway.

---

## Gemini-Specific & Antigravity Workflows

### 1. Active Antigravity Tool Suite Mapping & Safeguards
Antigravity utilizes the following specialized, fine-grained toolset for filesystem and system operations. Refer to this mapping and the mandatory operational safeguards below:

| Tool Category | Tool Name | Operational Guidance |
| :--- | :--- | :--- |
| **File Reading** | `view_file` | Read up to 800 lines at a time. Supports absolute paths. |
| **File Creation** | `write_to_file` | Create new files. Supports `IsArtifact` and structured `ArtifactMetadata` block. |
| **Surgical Edit** | `replace_file_content` | Replace a single contiguous block of code. Specify `StartLine`, `EndLine`, `TargetContent`, and `ReplacementContent` with 100% exact leading whitespace matching. |
| **Multi Edit** | `multi_replace_file_content` | Perform multiple non-contiguous edits within the same file simultaneously. Order chunks descendingly (bottom-to-top) to avoid line offsets. |
| **Search** | `grep_search` | Search codebases via Ripgrep. Keep `MatchPerLine: true` for line-by-line matches. Apply partitioning if matches exceed 50. |
| **Command Execution** | `run_command` | Execute PowerShell/Bash shell commands. Returns task process IDs. NEVER use `cd` commands. 🚫 **STRICT BAN**: NEVER run `git commit` or `git push` directly via this tool (e.g., using `$env:SYNC_ACTIVE=1; git commit` to bypass QA gates is FORBIDDEN). All commits must go through the approved `/sync` pipeline or `bun scripts/dev-sync.ts`. |

#### ⚠️ Surgical Multi-Replace Offset Safeguard
When calling `multi_replace_file_content` with multiple `ReplacementChunks`, the line numbers of subsequent target blocks will shift if previous edits change the line count.
- **Rule**: You **MUST** sort and process the `ReplacementChunks` from the **bottom of the file to the top** (descending order of line numbers: largest `StartLine` first).
- This guarantees that edits made near the end of the file do not alter or corrupt the line numbers of target blocks located higher up in the file.

#### ⚠️ Windows Terminal & Code Page Safeguard
When executing CLI commands via `run_command` on Windows (PowerShell/CMD), the default Windows code page (e.g., CP949) often causes Unicode decoding errors.
- **Rule:** Before running commands that output non-ASCII text, explicitly set the code page to UTF-8 by prepending `$OutputEncoding = [Console]::OutputEncoding = [System.Text.Encoding]::UTF8;` (PowerShell) or `chcp 65001` (CMD).

#### ⚠️ Grep Search 50-Match Cap Safeguard
The `grep_search` tool silently truncates results at exactly **50 matches**.
- **Rule**: If a codebase-wide search yields 50 results, do **NOT** assume you have all occurrences.
- **Remediation**: Partition the search. Divide the search by targeting specific subdirectories (e.g., `C:\git\<project>\src`) or apply restrictive file glob filters using the `Includes` parameter (e.g., `["*.py"]` or `["*.ts"]`).

---

### 2. Planning Mode & Artifact Specifications
Enter Planning Mode when:
- The user requests a new feature or significant refactor.
- The change modifies more than 2 files.
- The correct approach is unclear or contains architectural trade-offs.

When entering Planning Mode, Gemini **MUST** leverage the following three precise Markdown artifacts. When creating or updating them, set `IsArtifact: true` and specify accurate metadata:

#### 1. `implementation_plan.md` (Path: `<appDataDir>\brain\<session-id>\implementation_plan.md` on Windows / `<appDataDir>/brain/<session-id>/implementation_plan.md` on macOS/Linux)
*   **Purpose**: Detailed technical design document presented to the user for feedback and approval.
*   **Metadata**: `ArtifactType: "implementation_plan"`, `RequestFeedback: true`, and a clear multi-line `Summary`.
*   **Format Requirement**: MUST use the exact markdown template below for the document structure.
    ```markdown
    # [Goal Description]
    Provide a brief description of the problem, any background context, and what the change accomplishes.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [5throck/ai-workspace-standards](https://github.com/5throck/ai-workspace-standards) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
