---
trigger: always_on
description: Claude Code plugin marketplace with advanced context engineering techniques focused on improving agent result quality.
---

# Context Engineering Kit

Claude Code plugin marketplace with advanced context engineering techniques focused on improving agent result quality.

See @README for project overview and @CONTRIBUTING.md for contributing guidelines.

## Project Structure

```
context-engineering-kit/
├── .claude-plugin/
│   └── marketplace.json    # Main marketplace manifest with all plugins
├── plugins/                 # Plugin source code
│   └── <plugin-name>/
│       ├── .claude-plugin/
│       │   └── plugin.json  # Plugin manifest
│       ├── README.md
│       ├── commands/        # Slash commands (*.md)
│       └── skills/          # Skills (*.md)
├── docs/                    # Documentation (GitBook)
│   └── plugins/
│       └── <plugin-name>/   # Plugin documentation
│           └── README.md
├── specs/                   # Feature specifications
├── justfile                 # Development commands
└── CONTRIBUTING.md          # Contribution guidelines
```

## Available Plugins

code-review, customaize-agent, ddd, docs, git, kaizen, mcp, reflexion, sadd, sdd, tdd, tech-stack

## Development Commands

```bash
just help                                       # Show all commands
just list-plugins                               # List plugins with versions
just sync-docs-to-plugins                       # Copy docs/plugins/*/README.md → plugins/*/README.md
just sync-plugins-to-docs                       # Copy plugins/*/README.md → docs/plugins/*/README.md
just set-version <name> <x.y.z>                 # Update plugin version
just set-marketplace-version <x.y.z>            # Update marketplace version
```

## Key Development Rules

### Plugin Design Philosophy

1. **Commands over skills** - Commands load on-demand; skill descriptions load into context by default
2. **Specialized agents** - Use agents with focused context to reduce hallucinations
3. **Setup-commands** - Use setup commands to update CLAUDE.md for persistent project context
4. **Minimal tokens** - Every token counts; keep prompts concise

### When Creating/Modifying Plugins

- Use `just set-version <name> <x.y.z>` to update plugin versions consistently, do not modify manually.
- Use `just set-marketplace-version <x.y.z>` to update the marketplace version, do not modify manually.
- Keep README.md in sync between `plugins/<name>/` and `docs/plugins/<name>/` using `just sync-docs-to-plugins` and `just sync-plugins-to-docs` commands. Do not update both manually.
- Test plugins with Claude Code before committing using `plugins/customaize-agent:test-prompt` and `plugins/customaize-agent:test-skill` commands.

### When Adding New Skills or Commands

**Documentation Checklist** (all files must be updated):

1. `plugins/<name>/README.md` - Add skill/command with "Use when..." trigger and structured tables
2. `README.md` (root) - Add to Skills/Commands section under plugin listing
3. `docs/reference/skills.md` or `docs/reference/commands.md` - Add to complete reference
4. `docs/plugins/README.md` - Update Key Features for the plugin
5. `docs/resources/related-projects.md` - Add source project attribution if based on external work
6. `docs/resources/papers.md` - Add research papers if technique is based on academic research
7. Run `just sync-plugins-to-docs` to sync plugin README to docs/
8. Bump plugin version: `just set-version <name> <x.y.z>` (minor for features)
9. Bump marketplace version: `just set-marketplace-version <x.y.z>`

**Finding All References**: Before declaring documentation complete, search for all files referencing the plugin:

```bash
grep -r "<plugin-name>" docs/ README.md --include="*.md" -l
```

**Skill Documentation Pattern**:

- Start with "Use when..." trigger phrase
- Use tables for structured information (not prose)
- Include key concepts with one-line explanations
- Keep YAML `name:` field matching folder name for consistency

### When Creating/Refactoring Agents

**Agent File Location**: `.claude/agents/<agent-name>.md` or `plugins/<plugin>/agents/<agent-name>.md`

See `plugins/customaize-agent/commands/create-agent.md` command for detailed agent creation guidelines including frontmatter rules, required sections, process ordering, and decision table patterns.

## Use Context7 MCP for Loading Documentation

Context7 MCP is available to fetch up-to-date documentation with code examples.

**Recommended library IDs**:

- `/anthropics/claude-code` - Claude Code CLI tool documentation (1954 snippets)
- `/websites/platform_claude` - Claude Developer Platform comprehensive docs (5916 snippets)
- `/anthropics/anthropic-cookbook` - Code examples and guides for building with Claude (1226 snippets)
- `/anthropics/courses` - Anthropic educational courses on SDK and prompt engineering (1173 snippets)
- `/websites/platform_claude_en_agent-sdk` - Claude Agent SDK for Python/TypeScript (605 snippets)
- `/anthropics/claude-agent-sdk-python` - Python SDK for Claude Agent (57 snippets)
- `/anthropics/claude-code-sdk-python` - Python SDK for Claude Code (31 snippets)

**Usage**:

```
mcp__context7__query-docs libraryId: "/anthropics/claude-code" query: "how to configure hooks"
```

## Use Paper Search MCP for Academic Research


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NeoLabHQ/context-engineering-kit](https://github.com/NeoLabHQ/context-engineering-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
