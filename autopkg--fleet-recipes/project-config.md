---
trigger: always_on
description: This repository contains AutoPkg processors and recipes for Fleet device management platform integration. The main component is a Python processor that uploads software packages to Fleet and creates GitOps pull requests for configuration management.
---

# Fleet AutoPkg Recipes

This repository contains AutoPkg processors and recipes for Fleet device management platform integration. The main component is a Python processor that uploads software packages to Fleet and creates GitOps pull requests for configuration management.

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Communication Style

### Git Commit Messages and Pull Requests
- **NEVER use emojis** in commit messages or PR descriptions
- Use clear, professional language
- Focus on technical details and implementation specifics
- When creating PRs, always use `gh pr create --web` to open the browser for final review

### PR Creation Workflow
1. Create feature branch: `git checkout -b feature-name`
2. Stage changes: `git add <files>`
3. Commit with clear message (no emojis): `git commit -m "Description"`
4. Push branch: `git push -u origin feature-name`
5. Create PR with `--web` flag: `gh pr create --title "Title" --body "Description" --web`
6. The `--web` flag opens the browser for final review before submission

## Working Effectively

### Bootstrap and Setup (macOS required for AutoPkg)
- Install pre-commit for automated code style validation:
  - `python3 -m pip install --upgrade pip`
  - `python3 -m pip install pre-commit` -- takes 10-30 seconds. NEVER CANCEL.
  - `pre-commit install` -- installs git hooks
  - Note: The processor uses only native Python libraries available in AutoPkg's bundled Python (no external dependencies required for runtime)
- Install AutoPkg (macOS only):
  - Download the latest release from https://github.com/autopkg/autopkg/releases/latest
  - Install the `.pkg` file (e.g., `autopkg-2.7.3.pkg`)
  - Verify installation: `autopkg version`
  - Installation takes 1-2 minutes. NEVER CANCEL.
- Configure AutoPkg repositories:
  - `autopkg repo-add https://github.com/autopkg/recipes.git` -- takes 30-60 seconds. NEVER CANCEL.
  - `autopkg repo-add https://github.com/autopkg/homebysix-recipes.git` -- takes 30-60 seconds. NEVER CANCEL.
  - Note: This repository contains its own FleetImporter processor, so no additional repo is needed for Fleet integration
- Validate Python syntax and code style:
  - `pre-commit run --all-files` -- runs all code style checks (black, isort, flake8, recipe validation)

### Critical macOS Setup Notes
- AutoPkg ONLY works on macOS. Do not attempt to install on Linux/Windows.
- Install AutoPkg from GitHub releases: https://github.com/autopkg/autopkg/releases/latest
- Ensure Xcode Command Line Tools: `xcode-select --install`
- Python 3.9+ is required: `python3 --version`

### Environment Variables for Local Testing
- For Direct Mode (Fleet API):
  - `export FLEET_API_BASE="https://fleet.example.com"`
  - `export FLEET_API_TOKEN="your-fleet-api-token"`
  - `export FLEET_TEAM_ID="1"`
- For GitOps Mode (S3/GitHub):
  - `export AWS_S3_BUCKET="your-fleet-packages-bucket"`
  - `export AWS_CLOUDFRONT_DOMAIN="d1234567890abc.cloudfront.net"`
  - `export FLEET_GITOPS_REPO_URL="https://github.com/org/fleet-gitops.git"`
  - `export FLEET_GITOPS_GITHUB_TOKEN="your-github-token"`
  - `export FLEET_GITOPS_SOFTWARE_DIR="lib/macos/software"`
  - `export FLEET_GITOPS_TEAM_YAML_PATH="teams/workstations.yml"`
- Set `GIT_TERMINAL_PROMPT=0` to prevent interactive Git authentication prompts

### Testing and Validation
- Test AutoPkg Python has required modules: `/Library/AutoPkg/Python3/Python.framework/Versions/Current/bin/python3 -c "import yaml, urllib.request, json; print('Dependencies OK')"`
- Validate recipe YAML syntax: `python3 -c "import yaml; yaml.safe_load(open('GitHub/GithubDesktop.fleet.direct.recipe.yaml'))"`
- Test Git operations: `git status && git log --oneline -5`
- Run direct mode recipe locally (macOS only): `autopkg run GitHub/GithubDesktop.fleet.direct.recipe.yaml -v`
- Run gitops mode recipe locally (macOS only): `autopkg run GitHub/GithubDesktop.fleet.gitops.recipe.yaml -v`

### File Validation
- Always validate YAML syntax when modifying recipe files
- Pre-commit automatically runs on `git commit`, catching issues before they're committed
- Test environment variable substitution in recipes
- **ALWAYS write recipes in YAML format, not XML** - This repository uses YAML recipes exclusively

### Runtime Dependencies
- **The FleetImporter processor uses ONLY native Python libraries** - no external pip packages required at runtime
- AutoPkg's bundled Python 3.10+ includes all necessary modules: `urllib`, `json`, `yaml`, `hashlib`, etc.
- Development dependency (pre-commit) manages black, isort, flake8 automatically - no manual installation needed

## Code Style Requirements

ALL Python code must pass these checks before being committed:

### Pre-Commit Hooks (Automated)
The repository uses pre-commit hooks to automatically validate code style on every commit:
- **Black**: Consistent code formatting across the AutoPkg ecosystem
- **isort**: Standardized import organization
- **Flake8 with Bugbear**: Code quality, unused variables, style violations

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [autopkg/fleet-recipes](https://github.com/autopkg/fleet-recipes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
