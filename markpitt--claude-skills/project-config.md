---
trigger: always_on
description: This repository serves as a centralized store for custom Claude Skills that can be imported into Claude Code or Claude Desktop.
---

# Claude Skills Repository

This repository serves as a centralized store for custom Claude Skills that can be imported into Claude Code or Claude Desktop.

## Repository Structure

```
claude-skills/
├── CLAUDE.md              # This file - repository documentation
├── README.md              # User-facing documentation
├── skills/                # All skills stored here
│   └── skill-name/       # Individual skill directory
│       ├── SKILL.md      # Required: skill entry point
│       ├── resources/    # Optional: supporting files
│       ├── templates/    # Optional: forms or structured prompts
│       └── scripts/      # Optional: utility scripts
├── skills-tui/           # Skills TUI application (see below)
│   ├── src/              # Rust source code
│   ├── tests/            # Test suite
│   ├── Cargo.toml        # Rust package configuration
│   └── README.md         # TUI documentation
└── .claude/              # Claude Code configuration
    └── hooks/            # Repository hooks
```

## Skills TUI Application

The `skills-tui` directory contains a production-ready Terminal User Interface (TUI) application for browsing, filtering, and managing Claude Skills.

### Quick Start

```bash
cd skills-tui

# Build the project
cargo build --release

# Run the TUI
cargo run -- /path/to/skills
```

### Features

- **3-Pane Layout**: Navigate skills (left), view details (right), see status (bottom)
- **Installation**: Install skills to Claude Code, Claude Desktop, or download as ZIP
- **Search & Filter**: Find skills by name or description
- **Cross-Platform**: Works on Windows, macOS, and Linux
- **Test Coverage**: 12 comprehensive tests with 100% pass rate

### Keybindings

| Key | Action |
|-----|--------|
| `↑` / `↓` | Navigate skills |
| `i` | Install to Claude Code |
| `d` | Prepare for Claude Desktop |
| `z` | Download as ZIP |
| `f` | Toggle filter |
| `q` | Quit |

### Development

All functionality is built using **Test-Driven Development (TDD)**:

```bash
# Run all tests
cargo test

# Run specific test suite
cargo test --test discovery_test
cargo test --test install_test
cargo test --test zip_test
```

### Technology Stack

- **Ratatui 0.27**: Modern TUI framework
- **Crossterm 0.27**: Terminal backend
- **Serde**: YAML parsing for SKILL.md
- **Zip 0.6**: Archive creation
- **Walkdir**: Directory traversal

For detailed documentation, see [`skills-tui/README.md`](./skills-tui/README.md)

## Skill Structure

Each skill must follow this structure:

### Required Files
- `skills/skill-name/SKILL.md` - The main skill file with frontmatter and instructions

### Optional Directories
- `skills/skill-name/resources/` - Supporting documentation (reference.md, examples.md, etc.)
- `skills/skill-name/templates/` - Structured prompts or forms
- `skills/skill-name/scripts/` - Executable utilities (Python, Bash, etc.)

## SKILL.md Format

Every SKILL.md must include YAML frontmatter:

```markdown
---
name: skill-name
description: Brief description of what the skill does and when to use it (max 1024 chars)
allowed-tools: Read, Edit, Bash  # Optional: restrict available tools
version: 1.0
---

# Skill Name

Main skill instructions here...
```

### Frontmatter Fields
- `name` (required): Lowercase, hyphenated identifier (max 64 chars)
- `description` (required): Third-person description of purpose and when to use (max 1024 chars)
- `allowed-tools` (optional): Comma-separated list of tools Claude can use with this skill
- `disabled-model-invocation` (optional): Set to true to prevent Claude from making API calls
- `version` (optional): Semantic version number

## Best Practices

### Content Organization
- **Keep SKILL.md under 500 lines** - use referenced files for additional content
- **Use progressive disclosure** - only load details when needed
- **Write concise metadata** - descriptions should be clear and objective
- **Include examples** - help Claude understand expected outputs

### Performance
- Only name/description (~100 tokens) is pre-loaded into Claude's context
- Full SKILL.md is loaded when Claude determines the skill is relevant
- Additional resources are loaded only when referenced
- Include executable scripts for computationally-intensive operations

### Testing
- Test skills with representative tasks before committing
- Include minimal, deterministic test scripts when appropriate
- Use pre-commit hooks to validate skill structure

### Security
- Only reference trusted external resources
- Review all scripts before including
- Use allowed-tools to restrict capabilities when appropriate

## Common Bash Commands

### Testing Skills Locally
```bash
# Install skill to personal directory
cp -r skills/skill-name ~/.claude/skills/

# Install skill to project directory
cp -r skills/skill-name .claude/skills/

# Create a new skill
mkdir -p skills/new-skill
touch skills/new-skill/SKILL.md
```

### Building and Packaging
```bash
# Create zip for Claude Desktop
cd skills/skill-name
zip -r ../../skill-name.zip .

# Validate skill structure
test -f skills/skill-name/SKILL.md && echo "Valid" || echo "Missing SKILL.md"
```

### Git Operations
```bash
# Add new skill
git add skills/new-skill/
git commit -m "Add new-skill for [purpose]"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [markpitt/claude-skills](https://github.com/markpitt/claude-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
