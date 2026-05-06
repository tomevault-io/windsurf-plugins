---
trigger: always_on
description: Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.
---

# Best of Typst (TCDM) - GitHub Copilot Instructions

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

**Updated after main branch merge** - All commands and validation steps have been verified to work correctly in the current repository state.

## Working Effectively

### Bootstrap and Build
Run these commands in order to set up the development environment:

1. **Install system dependencies:**
   ```bash
   sudo apt-get update && sudo apt-get install -y just
   ```
   - Takes 2-3 minutes. NEVER CANCEL. Set timeout to 5+ minutes.

2. **Install Python dependencies:**
   ```bash
   just bootstrap
   ```
   - Installs `ruamel.yaml` Python package required for YAML processing
   - Takes ~3 seconds normally. Very fast and reliable with good network.
   - **May fail in sandboxed environments** with network restrictions (ReadTimeoutError from pypi.org)
   - **WORKAROUND**: If it fails, the dependency may already be installed - test with validation commands first

3. **Verify dependencies are working:**
   ```bash
   python -c "from ruamel.yaml import YAML; YAML(typ='safe').load('projects.yaml')"
   ```
   - Should run without error if dependencies are properly installed
   - Use this to test if ruamel.yaml is available before proceeding

4. **Install best-of generator (required for main functionality):**
   ```bash
   pip install "best-of @ git+https://github.com/YDX-2147483647/best-of-generator.git@best-of-bits"
   ```
   - Takes 20-25 seconds.
   - This is the core tool that generates README.md from projects.yaml

### Core Commands

#### List available commands:
```bash
just --list
```

#### Sync issue form templates:
```bash
just sync-issue-form
```
- Syncs categories and labels from projects.yaml to GitHub issue templates
- Takes <1 second. Always reliable.

#### Build for pandoc (creates processed markdown):
```bash
just build-for-pandoc
```
- Creates `build/index.md` from README.md for pandoc processing
- Takes <1 second. Creates build/ directory automatically.

#### Generate best-of list (main functionality):
```bash
best-of generate projects.yaml --libraries-key=$LIBRARIES_KEY --github-key=$GITHUB_TOKEN --gitee-key=$GITEE_API_KEY
```
- **CRITICAL**: This command may fail in sandboxed environments due to network restrictions (DNS resolution failures for api.npmjs.org, libraries.io, etc.)
- **WORKAROUND**: The command will run but with warnings and limited functionality without API keys
- Takes 2-3 seconds to fail with network issues, or 30+ seconds with proper network access
- **NEVER CANCEL**: If running with proper network access, allow up to 5+ minutes for completion

#### Add project from GitHub issue:
```bash
just add-project ISSUE_NUMBER
```
- Requires `gh` CLI and appropriate GitHub permissions
- Extracts project information from GitHub issue and adds to projects.yaml

#### List project suggestions:
```bash
just list-project-suggestions
```
- Lists GitHub issues with the "add-project" label
- Requires `gh` CLI access

## Validation

### Always run these validation steps before committing changes:

1. **Validate YAML syntax:**
   ```bash
   python -c "from ruamel.yaml import YAML; YAML(typ='safe').load('projects.yaml')"
   ```

2. **Validate YAML schema:**
   ```bash
   # Install boon (JSON Schema validator)
   wget https://github.com/santhosh-tekuri/boon/releases/download/v0.6.1/boon-x86_64-unknown-linux-gnu.tar.gz
   tar -xzf boon-x86_64-unknown-linux-gnu.tar.gz
   sudo mv boon /usr/local/bin/boon
   
   # Validate against schema
   boon scripts/projects.schema.json projects.yaml
   ```
   - Validates projects.yaml structure against the JSON schema
   - Ensures all required fields are present and types are correct
   - Checks category and label enum values are valid

3. **Check for duplicate project names:**
   ```bash
   yq '.projects.[] | line + " " + .name' projects.yaml | sort --key=2 | uniq --skip-fields=1 --all-repeated
   ```
   - Should return empty output. Any output indicates duplicate names.

4. **Sync issue forms after modifying projects.yaml:**
   ```bash
   just sync-issue-form
   ```

5. **Test build process:**
   ```bash
   just build-for-pandoc
   ```

### Manual Testing Scenarios

**CRITICAL**: After making changes to projects.yaml, always test these scenarios:

1. **Project addition workflow:**
   - Modify projects.yaml to add a new project
   - Run `just sync-issue-form` to update templates
   - Verify no YAML syntax errors
   - Check that new project appears in appropriate category

2. **Build verification:**
   - Run `just build-for-pandoc`
   - Verify `build/index.md` is created successfully
   - Check that markdown structure is preserved

## Repository Structure

### Key Files and Directories:
- `projects.yaml` - Main configuration file containing all projects and categories
- `justfile` - Build recipes and automation commands
- `scripts/` - Python automation scripts
- `.github/workflows/` - CI/CD pipelines for automation
- `config/header.md` - Template header for generated README
- `config/footer.md` - Template footer for generated README

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [YDX-2147483647/best-of-typst](https://github.com/YDX-2147483647/best-of-typst) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
