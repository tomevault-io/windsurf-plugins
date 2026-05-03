---
trigger: always_on
description: This repository contains reusable AI skills for **GitHub Copilot CLI**, **Claude Code**, and **OpenAI Codex**. Skills are Markdown-based workflow specifications (`SKILL.md`) that teach AI agents specific tasks.
---

# Copilot Instructions for CLI AI Skills

This repository contains reusable AI skills for **GitHub Copilot CLI**, **Claude Code**, and **OpenAI Codex**. Skills are Markdown-based workflow specifications (`SKILL.md`) that teach AI agents specific tasks.

## ⚠️ Important: Single Source of Truth

Skills are maintained in **`skills/`** directory (single source) and automatically synchronized to platform directories.

**DO NOT EDIT:**
- ❌ `.github/skills/` (auto-generated)
- ❌ `.claude/skills/` (auto-generated)
- ❌ `.codex/skills/` (auto-generated)

**DO EDIT:**
- ✅ `skills/` (source of truth)

After editing, run:
```bash
./scripts/build-skills.sh
# or
cd cli-installer && npm run build
```

## Build, Test, and Lint Commands

### NPM Package Commands (cli-installer)

```bash
# Run tests
cd cli-installer && npm test

# Link package locally for testing
cd cli-installer && npm link

# Unlink local package
cd cli-installer && npm unlink -g claude-superskills

# Generate skills index and catalog
cd cli-installer && npm run generate-all
# Or individually:
npm run generate-index    # Updates skills_index.json
npm run generate-catalog  # Updates CATALOG.md

# Version bump and publish workflow
./scripts/bump-version.sh [patch|minor|major]  # Updates version, commits, tags
./scripts/pre-publish-check.sh                 # Validates before publishing
```

### Validation Scripts

```bash
# Validate a single skill's YAML frontmatter (kebab-case naming, required fields)
./scripts/validate-skill-yaml.sh skills/<skill-name>

# Validate a single skill's content quality (word count, writing style)
./scripts/validate-skill-content.sh skills/<skill-name>

# Validate all skills
for skill in skills/*/; do
  ./scripts/validate-skill-yaml.sh "$skill"
  ./scripts/validate-skill-content.sh "$skill"
done

# Build skills (sync source to platforms)
./scripts/build-skills.sh

# Validate GitHub Actions workflows
./scripts/validate-workflows.sh

# Verify version consistency across package.json files
./scripts/verify-version-sync.sh
```

### Installation & Setup

```bash
# Check which AI tools are installed (gh copilot, claude)
./scripts/check-tools.sh

# Install skills globally via symlinks (updates automatically on git pull)
./scripts/install-skills.sh $(pwd)

# Create new skill scaffolding
./scripts/create-skill.sh <skill-name>
```

### Skill Installation via NPM (End Users)

The repository includes a CLI installer package (`cli-installer/`) that users can run to install skills:

```bash
# Zero-config installation (interactive)
npx claude-superskills

# Install specific bundle
npx claude-superskills --bundle essential -y    # skill-creator, prompt-engineer
npx claude-superskills --bundle content -y      # youtube-summarizer, audio-transcriber
npx claude-superskills --bundle developer -y    # skill-creator only
npx claude-superskills --bundle all -y          # All skills

# Install all skills
npx claude-superskills --all -y

# Search for skills
npx claude-superskills --search "prompt"

# List installed skills
npx claude-superskills list    # or: ls

# Update skills
npx claude-superskills update  # or: up

# Uninstall skills
npx claude-superskills uninstall <skill-name>  # or: rm

# Check installation health
npx claude-superskills doctor  # or: doc
```

Bundles are defined in `bundles.json` at repository root.

### Manual Testing

Test skills by installing them and running through trigger phrases:

```bash
# Install skills first
./scripts/install-skills.sh $(pwd)

# Test in new terminal session
gh copilot -p "improve this prompt: create REST API"

# Validate individual skill components
./scripts/validate-skill-yaml.sh .github/skills/prompt-engineer
./scripts/validate-skill-content.sh .github/skills/prompt-engineer
```

## Architecture

### Multi-Platform Design

Skills maintain **functional parity** across three platforms:

- **GitHub Copilot CLI** (`.github/skills/`) - Uses `view`, `edit`, `bash` tools
- **Claude Code** (`.claude/skills/`) - Uses `Read`, `Edit`, `Bash` tools
- **OpenAI Codex** (`.codex/skills/`) - Uses similar tool conventions

Only tool names and prompt prefixes differ (`copilot>`, `claude>`, `codex>`). Workflow logic is identical across all platforms.

### Skill Structure

Each skill is a directory containing:

```
skill-name/
├── SKILL.md           # Core specification with YAML frontmatter + workflow
├── README.md          # User-facing documentation
├── references/        # Optional: Detailed docs
├── examples/          # Optional: Working code samples
└── scripts/           # Optional: Executable utilities
```

### SKILL.md Anatomy

```yaml
---
name: kebab-case-name        # Required, lowercase with hyphens only
description: "This skill should be used when..."  # Required, third-person
triggers:                    # Recommended
  - "trigger phrase"
version: 1.0.0              # Required, SemVer
---

## Purpose
[What the skill does]

## When to Use
[Activation scenarios]

## Workflow
### Step 0: Discovery (if applicable)
[Runtime discovery of paths/values]

### Step 1: [Action]
[Instructions]

## Critical Rules
[NEVER/ALWAYS guidelines]

## Example Usage

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ericgandrade/claude-superskills](https://github.com/ericgandrade/claude-superskills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
