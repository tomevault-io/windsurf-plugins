---
trigger: always_on
description: - **Primary Repository**: `Framework-R-D/phlex`
---

# GitHub Copilot Instructions for Phlex Project

## Project Context & Workflow

### Repository Ecosystem

- **Primary Repository**: `Framework-R-D/phlex`
- **Design & Documentation**: `Framework-R-D/phlex-design` (contains design and other documentation)
- **Coding Guidelines**: `Framework-R-D/phlex-coding-guidelines` (coding guidelines for framework contributors)
- **Examples**: `Framework-R-D/phlex-examples` (example user code demonstrating Phlex usage)
- **Spack Recipes**: `Framework-R-D/phlex-spack-recipes` (Spack recipes for Phlex and dependencies)
- **Dependencies**: Critical dependency on `FNALssi/cetmodules` for the build system.
- **Container Images**:
  - `phlex-ci`: Used by automated CI checks.
  - `phlex-dev`: Used for VSCode devcontainers and local development.

### Codespace Layout

In a GitHub Codespace (or devcontainer), companion repositories are cloned
automatically alongside the primary repository:

- `/workspaces/phlex` — primary repository (workspace root)
- `/workspaces/phlex-design` — design documentation
- `/workspaces/phlex-examples` — example programs using Phlex
- `/workspaces/phlex-coding-guidelines` — coding guidelines for contributors
- `/workspaces/phlex-spack-recipes` — Spack recipes for Phlex and dependencies

Open `.devcontainer/codespace.code-workspace` to get a multi-root VS Code
window with all repositories visible. In VS Code: **File → Open Workspace from
File**, then select that file. From the terminal:

```bash
code /workspaces/phlex/.devcontainer/codespace.code-workspace
```

Git hooks are installed automatically when the devcontainer is first created
(`postCreateCommand` runs `prek install`). No manual setup is required.

### Development Workflow

- **Model**: Fork-based development. Developers should work on branches within their own forks.
- **Upstreaming**: Changes are upstreamed via Pull Requests (PRs) to the primary repository `Framework-R-D/phlex`.
- **Quality Standards**:
  - Adhere to design and coding guidelines in
    `Framework-R-D/phlex-design` and
    `Framework-R-D/phlex-coding-guidelines`, respectively.
  - Ensure code passes CI checks using the `phlex-ci` environment.
  - If you require changes to the `phlex-ci` or `phlex-dev` containers
    (or the Spack environments or auxiliary files they use), include
    those changes in the PR.
  - If an example in `phlex-examples` is rendered obsolete or invalid in
    some way, create an issue in the `Framework-R-D/phlex-examples`
    project if possible, explaining the conflict and likely changes
    required, and notify the user. If it is not possible to create an
    issue there, create one in the `phlex` repository if possible.
    Failing that, notify the user of the full details of the conflict.
  - If your changes require amendment/augmentation of documentation in
    `Framework-R-D/phlex-design`, create an issue there if possible, in
    `Framework-R-D/phlex` if not, or notify the user of details in the
    last resort.
  - If your changes require changes or additions to
    `Framework-R-D/phlex-spack-recipes`, (e.g. changes to dependency
    version requirements or new/removed dependencies), create an issue
    there if possible, in `Framework-R-D/phlex` if not, or notify the
    user of details in the last resort.
  - Minimize changes required for upstreaming.

### Git Branch Management

When creating branches for PRs:

- **Do not set upstream tracking at branch creation time**: When creating a new branch for eventual pushing as a new upstream branch (e.g., for a PR), it should not have an upstream tracking branch, even if created based on another branch
- **Rationale**: This eliminates the possibility of accidentally pushing commits to the base branch when pushing the new branch upstream
- **Best practice**: Create branches with `git checkout -b new-branch-name [<base-branch>]` or `git switch --no-track -c new-branch-name [<base-branch>]` without using `--track`, `-t`, or otherwise setting upstream
- **Push new branches**: Use `git push -u origin new-branch-name` only when ready to push the new branch to your fork for the first time; this is when you should set the upstream tracking branch

Example workflow:

```bash
# Create a new feature branch (no tracking)
git checkout -b feature/my-new-feature

# Make changes and commit
git add .
git commit -m "Add new feature"

# Push to your fork (sets tracking only now)
git push -u origin feature/my-new-feature
```

## Communication Guidelines

### Professional Colleague Interaction

Interact with the developer as a professional colleague, not as a subordinate:

- Avoid sycophancy and obsequiousness
- Point out mistakes or correct misunderstandings when necessary, using professional and constructive language
- If the developer's request contains an error or misunderstanding, explain the issue clearly

### Truth and Accuracy

Accuracy and honesty are critical:

- If you lack sufficient information to complete a task, say so explicitly: "I don't know" or "I don't have access to the information needed"
- Ask the developer for help or additional information when needed
- Never fabricate answers or hide gaps in knowledge
- It is better to acknowledge limitations than to provide incorrect information

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Framework-R-D/phlex](https://github.com/Framework-R-D/phlex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
