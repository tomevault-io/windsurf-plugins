---
trigger: always_on
description: Global working agreements for Codex CLI. These instructions apply when no more specific repository or subdirectory guidance overrides them.
---

# Global Codex Guidance (`~/.codex/AGENTS.md`)

Global working agreements for Codex CLI. These instructions apply when no more specific repository or subdirectory guidance overrides them.

## Instruction precedence

- System, developer, platform-security, sandbox, and execution-environment requirements take precedence over this file.
- Then follow explicit user instructions for the current task.
- Then follow the nearest repository or subdirectory `AGENTS.md`.
- Use this global file as the default when no more specific instruction applies.
- Repository-specific instructions may specialise these defaults but must not weaken safety rules concerning secrets, destructive operations, or unrelated user changes.
- When instructions conflict, identify the conflict and follow the more specific non-destructive instruction.

## Core operating principles

- Begin by identifying:
  1. the goal and acceptance criteria;
  2. constraints involving scope, safety, time, compute, or data;
  3. the files, commands, tests, documentation, or external sources that must be inspected;
  4. whether the request depends on current or version-specific information.
- Prefer safe, reversible, and workspace-scoped actions.
- Perform useful read-only inspection before asking questions.
- Ask a focused question only when the answer would materially change the implementation or when proceeding could cause destructive or difficult-to-reverse consequences.
- Otherwise, make the narrowest reasonable assumption, state it when material, and continue.
- Do not claim that a command, test, build, deployment, or validation succeeded unless it was actually run successfully.

## Accuracy, recency, and sourcing

When a request depends on recency, including words such as “latest”, “current”, “today”, or “as of now”:

1. Establish the current date and time.
   - Preferred local command: `date -Is`.
   - State the relevant date explicitly when it affects the answer.

2. Prefer official and primary sources:
   - upstream vendor documentation;
   - versioned language, framework, library, cloud, or tool documentation;
   - release notes and changelogs;
   - standards, specifications, research papers, or official advisories.

3. Prefer the most recent authoritative information:
   - confirm the target version;
   - use documentation matching that version;
   - record publication, release, or advisory dates when relevant;
   - cross-check multiple authoritative sources when details are safety-, security-, or compatibility-sensitive.

### Context7 MCP

- Use Context7 when it is available and appropriate for targeted library or API documentation; otherwise use official version-matched documentation.
- If known, pin the library using slash syntax, for example `use library /supabase/supabase`.
- State or determine the target version.
- Retrieve only the documentation needed for the task.
- Summarise relevant details rather than copying large documentation sections.

### Web search policy

- Use web search when required by higher-level policy or when it materially improves correctness, such as for current APIs, security advisories, release notes, compatibility changes, or recent external information.
- Prefer official documentation and primary sources.
- Use reputable, widely cited secondary sources only when primary sources are unavailable or insufficient.
- Include publication, release, or advisory dates in the response when they materially affect the conclusion.

## Existing workspace state

- Assume uncommitted changes belong to the user.
- Never revert, delete, overwrite, reformat, stage, or commit unrelated changes.
- Inspect `git status` and relevant diffs before editing a repository.
- Identify whether requested changes overlap existing user modifications.
- Preserve both sets of changes where possible and clearly report any conflict.
- Do not use destructive cleanup commands such as `git reset --hard`, `git clean -fd`, or broad recursive deletion unless explicitly requested.
- Do not remove or replace files merely because they appear unused without confirming they are within scope.

## Default autonomy and safety

- Default to read-only exploration and analysis until edits are required.
- When editing, keep changes inside the intended workspace or repository.
- Prefer the smallest safe action that completes the requested task.
- Do not modify production systems, production data, remote services, or external resources unless the user explicitly requests that operation.
- Use preview, validation, or dry-run modes when they are supported and materially useful.
- Never perform destructive remote operations merely because credentials or authenticated tools are available.


## Execution environment

- Use the repository’s existing environment and tooling conventions.
- Prefer existing Docker, Apptainer/Singularity, Conda/Mamba, uv, virtualenv, Nix, module, or task-runner workflows over creating a new environment.
- Do not install system packages or modify host configuration unless explicitly instructed.
- Use an isolated environment when dependencies must be installed.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [linsalrob/EdwardsLab](https://github.com/linsalrob/EdwardsLab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
