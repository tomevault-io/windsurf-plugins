---
trigger: always_on
description: When working on this repository, you must adhere to the following strict guidelines:
---

# Agent Coding Standards and Security Guidelines

When working on this repository, you must adhere to the following strict guidelines:

## 1. Workflow Architecture and Script Coding Standards
- **Do not embed scripts in workflows.** All GitHub Actions workflows must reference external scripts (e.g., in `.github/scripts/`) rather than embedding inline script logic. This ensures extensibility, reusability, testability, and automated linting.
- **Single Responsibility.** Each script in `.github/scripts/` must fulfill a focused operational step in the CI/CD pipeline.
- **Prevent Over-Fragmentation & Script Sprawl.** Modularity must not degrade into excessive micro-scripts. Common operational utilities (such as rendering step summaries via `generate-workflow-summary.py` or discovering files across operational modes via `find-files.py`) must be unified into cohesive, domain-scoped modules with clear CLI interfaces (`--workflow`, `--step`, `--all`) rather than duplicating near-identical one-off scripts.
- **Standardized Script Architecture.** All Python scripts in `.github/scripts/` must adhere to a consistent, modular structural approach:
  1. **Header & Docstring**: Declare the following 4 structured sections in every script:
     - `Operational Purpose:` Concise description of the operational task and CI/CD context.
     - `Required Environment Variables:` Explicit list of consumed environment variables or `None.`
     - `Outputs:` Explicit list of generated file artifacts, exported environment variables, or `$GITHUB_OUTPUT` keys.
     - `JSON Artifact Dependencies:` Specific configuration artifacts consumed from `.github/artifacts/` or `None.`
  2. **Imports**: Group imports cleanly: Standard Library -> Third-party -> Typing.
  3. **Configuration Loader**: Encapsulate configuration retrieval within a dedicated loader (`ConfigLoader` class or `load_config()` function) that strictly loads from `.github/artifacts/` with fail-fast validation.
  4. **Domain Logic**: Implement testable, modular functions and classes with type hints and comprehensive error handling.
  5. **Standardized Process Entrypoint**: Execute under `if __name__ == "__main__":` and exit cleanly using standard exit codes:
     - `0`: Success.
     - `1`: General or operational failure with actionable diagnostic logs.
     - Documented specialized codes (e.g., `EXIT_QUOTA_EXHAUSTED = 2` for pipeline retry control).

## 2. Configuration and Artifact Standards
- **No Hardcoded Values.** Scripts must never hardcode configuration values, thresholds, retry counts, time delays, model names, prompt paths, label names, or operational settings.
- **Use JSON Artifacts.** All configuration must be loaded from JSON artifacts located in `.github/artifacts/` (e.g., `workflow-config.json`, `ai-models.json`, `report-categories.json`, `discovery-feedback.json`, `readme-updater-config.json`, `google-search-config.json`).
- **Standardized Artifact Architecture.** All JSON artifacts in `.github/artifacts/` must adhere to uniform structural principles:
  1. **Self-Documenting Schema**: Include `_comment` or `comment` attributes on complex sections or thresholds explaining operational rationale.
  2. **Semantic Hierarchy**: Organize settings into logical sub-objects by pipeline task or domain (e.g., `workflow.discovery`, `workflow.conversion`, `configurations.<task>`, `task_models.<task>`).
  3. **Explicit Numeric Units**: Encode time, sizes, and counts explicitly in key names (e.g., `_seconds`, `_mb`, `_days`, `_bytes`, `_multiplier`, `_limit`).
  4. **POSIX Path Conventions**: All file and prompt paths referenced within artifacts must use forward slashes (e.g. `.github/ai-prompts/foo.md`) for seamless cross-platform execution.
- **Fail Fast on Missing Config.** If a required JSON artifact or configuration key is missing or invalid, scripts must raise an explicit exception (`ValueError`, `FileNotFoundError`, or `KeyError`) with descriptive remediation advice. Never fall back to unversioned, hardcoded default literals in script source code.

## 3. AI Prompts and Instruction Sets
- **Extract all prompts.** Any AI calls that rely on a prompt, instruction set, schema description, retry guidance, or fallback instructions must never define prompt text inline within code.
- **Use Markdown for Prompts.** All prompt text must be extracted into standalone `.md` files in `.github/ai-prompts/`.
- **Dynamic Prompt Paths.** Prompt filepaths must be declared in `workflow-config.json` and loaded dynamically by scripts at runtime.
- **Standardized Prompt Architecture.** All prompt markdown files in `.github/ai-prompts/` must adhere strictly to the repository's standardized instruction set structure:
  1. `# AI Instruction Set for <Task Name>` (H1 Title)
  2. `## Purpose` (Concise operational intent of the AI step)
  3. `## Goals` (Numbered list of specific objectives and non-functional constraints)
  4. `## Instructions` / `## Conversion Instructions` / `## Extraction Instructions` / `## Retry Instructions` (Detailed operational rules, negative constraints, and output formatting)
  5. `### Example Output` (Representative demonstration showing expected output structure where applicable)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jacobdjwilson/awesome-annual-security-reports](https://github.com/jacobdjwilson/awesome-annual-security-reports) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
