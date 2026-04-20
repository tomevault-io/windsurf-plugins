---
trigger: always_on
description: **Official Anthropic documentation (always check these first):**
---

# Contributing Skills

## Resources

**Official Anthropic documentation (always check these first):**

- [Claude Code Plugins](https://code.claude.com/docs/en/plugins)
- [Agent Skills](https://code.claude.com/docs/en/skills)
- [Best Practices](https://code.claude.com/docs/en/skills#best-practices)
- [Skill Authoring Best Practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices) — progressive disclosure, degrees of freedom, workflow checklists
- [The Complete Guide to Building Skills](https://resources.anthropic.com/hubfs/The-Complete-Guide-to-Building-Skill-for-Claude.pdf) ([text](https://gist.github.com/liskl/269ae33835ab4bfdd6140f0beb909873)) — evaluation-driven development, iterative testing

**Reference skills** - learn by example at different complexity levels:

| Complexity | Skill | What It Demonstrates |
|------------|-------|---------------------|
| **Basic** | [ask-questions-if-underspecified](plugins/ask-questions-if-underspecified/) | Minimal frontmatter, simple guidance |
| **Intermediate** | [constant-time-analysis](plugins/constant-time-analysis/) | Python package, references/, language-specific docs |
| **Advanced** | [culture-index](plugins/culture-index/) | Scripts, workflows/, templates/, PDF extraction, multiple entry points |

**When in doubt, copy one of these and adapt it.**

**Deep dives on skill authoring:**
- [Claude Skills Deep Dive](https://leehanchung.github.io/blogs/2025/10/26/claude-skills-deep-dive/) - Comprehensive analysis of skill architecture

**Example plugins worth studying:**
- [superpowers](https://github.com/obra/superpowers) - Advanced workflow patterns, TDD enforcement, multi-skill orchestration
- [compound-engineering-plugin](https://github.com/EveryInc/compound-engineering-plugin) - Production plugin structure
- [getsentry/skills](https://github.com/getsentry/skills) — Production Sentry skills; `security-review` is a standout routing + progressive disclosure example

**For Claude:** Use the `claude-code-guide` subagent for plugin/skill questions - it has access to official documentation.

## Technical Reference

### Codex Compatibility

This repository now supports both Claude plugin discovery and Codex-native skill discovery.

Rules:

- If a plugin adds `skills/<name>/SKILL.md`, it must also remain reachable through `.codex/skills/<name>`.
- If a plugin is command/hook/agent-only and has no `skills/` directory, add an explicit Codex wrapper skill under `.codex/skills/<plugin-name>/SKILL.md` or document why no Codex equivalent is intended.
- Before submitting, run:

```sh
python3 .github/scripts/validate_codex_skills.py
```

- If this check fails in CI, the remediation path should be local and mechanical: run the installer or update the `.codex/skills/` mapping, then commit the resulting changes.

### Plugin Structure

```
plugins/
  <plugin-name>/
    .claude-plugin/
      plugin.json         # Plugin metadata (name, version, description, author)
    commands/             # Optional: slash commands
    agents/               # Optional: autonomous agents
    skills/               # Optional: knowledge/guidance
      <skill-name>/
        SKILL.md          # Entry point with frontmatter
        references/       # Optional: detailed docs
        workflows/        # Optional: step-by-step guides
        scripts/          # Optional: utility scripts
    hooks/                # Optional: event hooks
    README.md             # Plugin documentation
```

**Important**: Component directories (`skills/`, `commands/`, `agents/`, `hooks/`) must be at the plugin root, NOT inside `.claude-plugin/`. Only `plugin.json` belongs in `.claude-plugin/`.

### Frontmatter

```yaml
---
name: skill-name              # kebab-case, max 64 chars
description: "Third-person description of what it does and when to use it"
allowed-tools:                # Optional: restrict to needed tools only
  - Read
  - Grep
---
```

### Naming Conventions

- **kebab-case**: `constant-time-analysis`, not `constantTimeAnalysis`
- **Gerund form preferred**: `analyzing-contracts`, `processing-pdfs` (not `contract-analyzer`, `pdf-processor`)
- **Avoid vague names**: `helper`, `utils`, `tools`, `misc`
- **Avoid reserved words**: `anthropic`, `claude`

### Path Handling

- Use `{baseDir}` for paths, **never hardcode** absolute paths
- Use forward slashes (`/`) even on Windows

### Python Scripts

When skills include Python scripts with dependencies:

1. **Use PEP 723 inline metadata** - Declare dependencies in the script header:
   ```python
   # /// script
   # requires-python = ">=3.11"
   # dependencies = ["requests>=2.28", "pydantic>=2.0"]
   # ///
   ```

2. **Use `uv run`** - Enables automatic dependency resolution:
   ```bash
   uv run {baseDir}/scripts/process.py input.pdf
   ```

3. **Include `pyproject.toml`** - Keep in `scripts/` for development tooling (ruff, etc.)

4. **Document system dependencies** - List non-Python deps (poppler, tesseract) in workflows with platform-specific install commands

### Hooks

PreToolUse hooks run on every Bash command—performance is critical:

- **Prefer shell + jq** over Python—interpreter startup (Python + tree-sitter) adds noticeable latency

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [trailofbits/skills](https://github.com/trailofbits/skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
