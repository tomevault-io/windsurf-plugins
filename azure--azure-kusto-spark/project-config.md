---
trigger: always_on
description: This skill has **two independent workflows** that can be invoked separately:
---

# SKILL: Azure Kusto Spark Connector — Release Process

## Identity

You are a release automation agent for the Azure Kusto Spark Connector. You execute the complete release lifecycle: cherry-picking changes between branches, bumping versions, updating the changelog, creating tags, and triggering the release pipeline. You operate by running git and shell commands in the repository.

## How to Invoke This Skill

This file is **not auto-loaded** by AI agents. You must explicitly reference it when prompting. The file must be present in your working directory (i.e., merged to the branch you're working from).

### Copilot CLI (Terminal)

Use the `@` prefix to include this file in your prompt:

**Development Workflow:**
```
@SKILL.md I have a branch with changes ready. Take over and get it merged to both branches.
```

```
@SKILL.md My PR #485 was just merged to master. Cherry-pick to Spark 3.
```

**Release Workflow:**
```
@SKILL.md All features are merged. Do a release.
```

```
@SKILL.md Bump the version and release. Both branches already have the latest changes merged.
```

```
@SKILL.md Just tag and release version 7.0.6 on both branches.
```

### VS Code — GitHub Copilot Chat

Use `#file:SKILL.md` to reference this file, or `@workspace` to let Copilot find it:

**Development Workflow:**
```
#file:SKILL.md My PR was just merged to master (commit abc1234). Cherry-pick to release/spark3.
```

```
@workspace Follow SKILL.md. I have changes on branch asaharn/feat/my-feature ready to push.
```

**Release Workflow:**
```
#file:SKILL.md Bump version to 7.0.6 and update the changelog on both branches.
```

### JetBrains — AI Assistant

Reference the file in AI Assistant chat:

```
Look at SKILL.md in the project root. My PR #485 was merged to master. Cherry-pick it to release/spark3.
```

```
Follow the instructions in SKILL.md to do a release — bump version, update changelog, and tag both branches.
```

### GitHub.com — Copilot in PR / Issues

When using Copilot on GitHub.com (e.g., in a PR comment or issue), reference the file path:

```
Follow the development workflow in SKILL.md at the repo root. This PR is now merged — describe the next steps to cherry-pick to release/spark3.
```

> **Note**: GitHub.com Copilot cannot execute git commands directly. It will provide the commands for you to run, or guide you through the GitHub UI steps.

### Prompt Tips

- **Be specific about where you are**: "My PR is merged", "I have a branch ready", "Both branches are updated"
- **Provide commit SHAs or PR numbers** when starting from Phase 2 (cherry-pick)
- **Specify a version** if you don't want auto-detection (e.g., "release version 7.0.6")
- **The agent will ask** for missing info — you don't need to provide everything upfront

## How to Use This Skill

This skill has **two independent workflows** that can be invoked separately:

1. **Development Workflow** (Phases 1–2) — Repeat for each feature/fix. Gets changes onto both branches.
2. **Release Workflow** (Phases 3–5) — Run once when ready to release. Bundles all changes since the last tag.

You can invoke either workflow at any point. Tell the agent where you are and it will pick up from there.

### Example Prompts — Development Workflow

**"I have a branch with changes ready — take over from here"**
> The agent will detect your current branch, push it if needed, help create a PR to master, and then cherry-pick to `release/spark3`.

**"My PR was just merged to master. Cherry-pick it to Spark 3"**
> Provide the merged PR number or the squash merge commit SHA. The agent will create a branch from `release/spark3`, cherry-pick, push, and create a PR.

### Example Prompts — Release Workflow

**"All features are merged. Do a release"**
> The agent will run Phases 3–5: bump the POM version on both branches, update the changelog with all changes since the last tag, create tags, and trigger the release pipeline.

**"Bump the version and update changelog for release"**
> The agent will start at Phase 3 — bump the POM version, update the changelog, then ask for confirmation before tagging.

**"Just tag and release version X.Y.Z"**
> The agent will start at Phase 5 — verify the version in `pom.xml`, create tags on both branches, and trigger the release.

### What the Agent May Ask You

If information is missing, the agent should ask for:

| Missing Info | When Needed |
|---|---|
| Squash merge commit SHA | Development Workflow — Phase 2 (cherry-pick) |
| Version number (if not auto-detecting) | Release Workflow — Phase 3+ |
| Change descriptions for changelog | Release Workflow — Phase 4, if not derivable from PRs |
| Confirmation before pushing tags | Release Workflow — always before Phase 5 |

## Repository Context

| Item | Value |
|---|---|
| Repository | `Azure/azure-kusto-spark` |
| Spark 4 branch | `master` |
| Spark 3 branch | `release/spark3` |
| Spark 4 tag format | `v4.0_{version}` (e.g., `v4.0_7.0.5`) |
| Spark 3 tag format | `v3.0_{version}` (e.g., `v3.0_7.0.5`) |
| Version property | `<revision>` in root `pom.xml` |
| CI workflow | `.github/workflows/build.yml` — runs on all pushes and PRs |
| Release workflow | `.github/workflows/release.yml` — triggers on tags matching `v[34].0_*` or via manual dispatch |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Azure/azure-kusto-spark](https://github.com/Azure/azure-kusto-spark) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
