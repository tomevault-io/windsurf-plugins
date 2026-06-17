---
trigger: always_on
description: Automated code review pipeline powered by OpenCode.
---

# opencode-reviewer

Automated code review pipeline powered by OpenCode.

## Architecture

```
cmd/reviewer/main.go         → CLI entry point (kong + TOML config)
cmd/comment-warrior/main.go → GitLab discussion follow-up CLI
internal/review/             → Review parsers and review-specific packages
internal/review/pipeline/    → Review pipeline orchestration
internal/review/runtime/     → Review OpenCode runtime loading and built-in tools
internal/review/agentsmd/    → AGENTS.md & CLAUDE.md swap (empty for review)
internal/review/agentconfig/ → Reviewer agent prompt loading
internal/review/finalizerconfig/ → Finalizer prompt/message loading
internal/review/promptconfig/ → Reviewer message loading
internal/commentwarrior/     → GitLab discussion classification and task logic
internal/commentwarrior/runtime/ → Comment-warrior OpenCode runtime and built-in tool
internal/shared/config/      → Configuration loading (TOML + ENV + config-dir defaults)
internal/shared/git/         → Git operations and repository preparation
internal/shared/diff/        → Diff parsing, filtering, context file generation
internal/shared/runner/      → OpenCode serve/run management
internal/shared/providerconfig/ → Provider JSON loading
internal/shared/subagentconfig/ → Sub-agent prompt loading
internal/shared/workspace/   → Generic temporary workspace builder for opencode config
internal/shared/vcs/         → VCS publisher interface, line normalizer, Markdown formatting
internal/shared/vcs/gitlab/  → GitLab MR comments publisher (REST API client)
configs/                     → TOML configs, provider.json, agent-prompt.md
prompt-examples/             → Example prompt files for parallel review sessions
prompts/                     → Prompt templates
```

## Configuration

TOML config file (`configs/example.toml`) with sections:

| Section      | Key                    | Description                                           |
|--------------|------------------------|-------------------------------------------------------|
| (root)       | `project_dir`          | Absolute path to the project repository (required)    |
| `[env]`      | `KEY = "VALUE"`        | Env vars: override TOML fields, but not system env vars |
| `[opencode]` | `endpoint`             | URL of running opencode serve (optional)              |
| `[opencode]` | `port`                 | Port for opencode subprocess (default: 4096)          |
| `[opencode]` | `model`                | LLM model identifier                                  |
| `[opencode]` | `binary`               | Path to opencode binary (default: opencode)           |
| `[opencode]` | `stage_timeout`        | Max seconds per review stage (default: 600)           |
| `[opencode]` | `max_steps`            | Max agent steps per session (default: 50)             |
| `[opencode]` | `min_version`          | Minimum required opencode version (semver)            |
| `[opencode]` | `provider_config_path` | Path to provider JSON config (relative to TOML file)  |
| `[git]`      | `remote`               | Git remote name (default: origin)                     |
| `[git]`      | `branch`               | Branch to review                                      |
| `[git]`      | `base_branch`          | Base branch for diff (default: main)                  |
| `[pipeline]` | `review_agent_prompt_path` | Path to reviewer agent prompt file (relative to TOML)    |
| `[pipeline]` | `review_agent_prompt`      | Inline reviewer agent prompt (alternative to path)       |
| `[pipeline]` | `review_message_paths`     | List of reviewer message files; each triggers a parallel session |
| `[pipeline]` | `review_messages`          | Inline reviewer messages (alternative to paths)          |
| `[pipeline]` | `finalizer_prompt_path`    | Path to finalizer agent prompt file (relative to TOML)   |
| `[pipeline]` | `finalizer_prompt`         | Inline finalizer agent prompt (alternative to path)      |
| `[pipeline]` | `finalizer_message_path`   | Path to finalizer user message file (relative to TOML)   |
| `[pipeline]` | `finalizer_message`        | Inline finalizer user message (alternative to path)      |
| `[pipeline]` | `review_sub_agent_prompt_paths`    | Paths to reviewer sub-agent prompt files (relative to TOML)     |
| `[pipeline]` | `review_sub_agent_prompts`         | Inline reviewer sub-agent prompts (alternative to paths)        |
| `[pipeline]` | `finalizer_sub_agent_prompt_paths` | Paths to finalizer sub-agent prompt files (relative to TOML)    |
| `[pipeline]` | `finalizer_sub_agent_prompts`      | Inline finalizer sub-agent prompts (alternative to paths)       |
| `[gitlab]`   | `url`                      | GitLab instance URL (e.g. https://gitlab.example.com)    |
| `[gitlab]`   | `token`                    | GitLab private access token                              |
| `[gitlab]`   | `project_id`               | Numeric GitLab project ID                                |
| `[gitlab]`   | `clear_comments`           | Delete open MR discussions before posting (default: false) |
### Environment Variables

Use config-dir or TOML for file-based configuration. Environment variables override scalar TOML settings; provider and prompt env vars are deprecated fallbacks when config-dir is inactive.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aaudin90/opencode-reviewer](https://github.com/aaudin90/opencode-reviewer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
