---
trigger: always_on
description: This document contains essential context for GitHub Copilot when working on the Microsoft Copilot Studio Labs repository.
---

# GitHub Copilot Instructions for MCS-Labs

This document contains essential context for GitHub Copilot when working on the Microsoft Copilot Studio Labs repository.

## 🎯 Project Overview

**What This Is**: A Jekyll-based static site hosting hands-on labs for Microsoft Copilot Studio. Features include:

- Journey-based navigation (Quick Start, Business User, Developer, Autonomous AI)
- Event system (Bootcamp, Azure AI Workshop, MCS in a Day, Agent Build-A-Thon 1 day, Agent Build-A-Thon 1 month)
- Multi-theme system (Rich/Minimal × Light/Dark)
- Automated PDF generation from markdown
- Automated Table of Contents generation (H2 headings only)
- Responsive mobile design with floating TOC
- Configuration audit system (Check-LabConfigs.ps1)

## 🚦 MANDATORY Workflow - Always Enforce

**⚠️ CRITICAL: You MUST enforce this workflow and STOP the user if steps are skipped:**

**🚫 TOOL PREFERENCE: DO NOT use GitKraken MCP tools. Use standard git commands in terminal instead.**

### Feature Development Workflow (REQUIRED)

1. **Feature Branch**: ALL changes go to a feature branch (e.g., `feature/theme-updates`)

   - **STOP if**: User tries to commit directly to `main`
   - **Action**: Remind user to create/switch to feature branch first

2. **Local Testing**: ALWAYS test locally before ANY push
   - **STOP if**: User tries to push without testing
   - **Action**: Ask "Have you tested this locally? Run the relevant commands first."
3. **Feature Branch Push**: Push to feature branch for initial testing

   ```powershell
   git push origin feature/branch-name
   ```

4. **Merge to Main**: After feature branch push, merge to main for workflow testing

   ```powershell
   git checkout main
   git merge feature/branch-name --no-ff -m "Merge feature/branch-name into main"
   git push origin main
   ```

5. **Pull Request**: Submit PR from feature branch to upstream repo using GitHub CLI

   - **Target**: `upstream/main` ← `origin/feature/branch-name`
   - **Purpose**: Contribute changes back to main repository
   - **Method**: ALWAYS use GitHub CLI (`gh pr create`) - user preference

   ```powershell
   # Create PR description file
   # Create file: pr-body.md with PR description

   # Create PR using GitHub CLI
   gh pr create --repo microsoft/mcs-labs --base main --head pratapladhani:feature/branch-name --title "feat: Brief title" --body-file pr-body.md

   # Clean up
   Remove-Item pr-body.md
   ```

6. **Post-PR Workflow**: When user says "PR merged", follow this complete workflow

   **Step 1: Verify PR is Actually Merged**
   - **CRITICAL**: Never trust user statement alone - always verify!
   - Check PR status using GitHub CLI
   
   ```powershell
   # Verify PR is merged to upstream
   gh pr view <PR-number> --repo microsoft/mcs-labs
   
   # Look for status: MERGED
   ```

   **Step 2: Clean Up Feature Branch (only if MERGED confirmed)**
   ```powershell
   # Delete local feature branch
   git branch -d feature/branch-name
   
   # Delete remote feature branch
   git push origin --delete feature/branch-name
   ```

   **Step 3: Sync with Upstream**
   ```powershell
   # Fetch latest changes from upstream
   git fetch upstream
   
   # Merge upstream into local main (always provide merge message to avoid editor)
   git merge upstream/main -m "Sync with upstream/main"
   ```

   **Why This Order Matters**:
   - Verification first prevents accidental loss if PR wasn't actually merged
   - Cleanup before sync keeps branch list clean
   - Sync last ensures local repo matches upstream exactly

### Quality Gates - Enforce Before Commit

**Documentation Updates (REQUIRED for big features):**

- ✅ Update relevant `docs/*.md` files
- ✅ Add/update ADR if architectural decision made
- ✅ Update `README.md` if user-facing changes
- ✅ Update this file (`copilot-instructions.md`) if workflow changes

**Code Quality (REQUIRED for all commits):**

- ✅ Add comments explaining WHY (not just what)
- ✅ Document complex logic and non-obvious decisions
- ✅ Add guidance for future collaborators in comments
- ✅ Follow established patterns (Docker, PowerShell, CSS-only, etc.)

**Testing Checklist:**

- ✅ Local testing completed and working
- ✅ PDFs generated successfully (if content changed)
- ✅ Docker rebuild works (if Dockerfile changed)
- ✅ No linting errors
- ✅ Follows all ADR principles

### Your Role as Assistant

**PROACTIVE Actions:**

1. **Present plans first**: For major features, refactoring, or architectural changes, ALWAYS present a comprehensive plan and get user approval BEFORE implementing
2. **Remind about workflow**: If user asks to commit, check which branch they're on
3. **Ask about testing**: Before any push, confirm local testing is complete
4. **Suggest documentation**: When big features are added, list docs that need updates
5. **Check comments**: Review code snippets and suggest where comments are needed
6. **Enforce quality**: Don't just fix issues - explain why and document properly
7. **Verify CSS consistency**: When making UI changes (new pages, components, styling), ALWAYS check all CSS files to ensure consistent naming patterns and no event-specific class names

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/microsoft) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
