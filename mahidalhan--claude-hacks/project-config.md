---
trigger: always_on
description: USE WHEN: trigger conditions, user phrases, task types that match.
---

# Claude Hacks

Skills, commands, and hacks for Claude Code. Install only what you need.

## Structure

```
claude-hacks/
├── .claude-plugin/
│   └── marketplace.json        # Single source of truth for all plugins
├── skills/
│   └── {skill-name}/           # Each skill is its own installable plugin
│       └── skills/
│           └── {skill-name}/
│               └── SKILL.md    # Skill definition (auto-invocable)
├── commands/
│   └── {command-name}/         # Standalone commands (no skill)
│       └── commands/
│           └── {command}.md
├── README.md
└── CLAUDE.md
```

## Available Plugins

### Skills (auto-invocable by Claude)
Skills are now directly invocable by Claude Code based on their description—no wrapper commands needed.

| Skill | Trigger Phrases | Description |
|-------|-----------------|-------------|
| standup | "status", "brief me", "catch up" | Project status briefing from git/PRs/issues |
| exa-code-context | "find examples", "code search" | Search GitHub for real code examples |
| architecture-introspector | "analyze architecture", "review design" | First-principles architecture analysis |
| explainer | "explain", "teach me", "how does X work" | Interactive chunked explanations |
| threaded-explainer | "deep dive", "explain in threads" | Deep-dive nested topic threads |
| mermaid-diagrams | "diagram", "visualize", "flowchart" | Publication-quality Mermaid diagrams |
| frontend-design | "design UI", "build interface" | Distinctive UI design |
| algorithmic-art | "generative art", "create art" | Procedural generative art |
| ascii-art-explainer | "ASCII diagram", "terminal visual" | ASCII art visualizations |
| xml-context-engineering | "structure context", "XML tags" | XML context structuring for LLMs |
| task-orchestrator | "orchestrate", "plan tasks" | Multi-step task planning |
| skill-creator | "create skill", "new skill" | Scaffold new skills |

### Commands (user-invoked via `/command`)
| Plugin | Commands | Description |
|--------|----------|-------------|
| git-commit | /commit | Create git commits for session changes |
| describe-pr | /describe-pr | Generate PR descriptions |
| handoff | /create-handoff, /resume-handoff | Session continuity |
| plan-code-changes | /plan-code-changes | Implementation planning |
| deslop | /deslop | Remove over-engineering |
| rich-simplicity | /rich-simplicity | Simple Made Easy review |
| test-coverage | /test-coverage | Test coverage analysis |

<ultrathink_guide>
## ULTRATHINK: Deep Thinking Mode

Claude Code supports extended thinking with trigger words that allocate more reasoning tokens:
- **"think"** → baseline thinking budget
- **"think hard"** → increased budget
- **"think harder"** → more budget
- **"ultrathink"** → maximum budget (31,999 tokens)

### When to Use ULTRATHINK

Best for:
- Architectural decisions with multi-system tradeoffs
- Complex debugging after 2+ failed attempts
- System-wide analysis spanning 10+ components
- Critical migrations and legacy system work
- Multi-step orchestration with non-obvious dependencies

NOT for:
- Simple code questions
- Bug fixes with obvious solutions
- Tasks that don't require deep reasoning
- Every request (wastes tokens/cost)

### Skills with ULTRATHINK Support

These skills benefit from deep thinking:
- **architecture-introspector** - First-principles analysis across all 6 phases
- **task-orchestrator** - Comprehensive dependency graph mapping
- **xml-context-engineering** - Thorough attention economics analysis

Example usage:
```
ultrathink and analyze the architecture of this codebase
think harder about the task dependencies here
```
</ultrathink_guide>

<critical_learnings>
## Maintenance Rules

### Flat Structure (CRITICAL)
Each skill/command is its own installable plugin. Users install individual capabilities, not bundles.

**Adding a new skill:**
1. Create `skills/{skill-name}/skills/{skill-name}/SKILL.md`
2. Add entry to marketplace.json
3. Commit and push

**Adding a standalone command:**
1. Create `commands/{command-name}/commands/{command}.md`
2. Add entry to marketplace.json
3. Commit and push

### Version Bumping (CRITICAL)
Claude Code caches plugins by version. Changes are INVISIBLE until version is bumped.

**When to bump version:**
- Any content changed in SKILL.md or command files
- Any file added or removed

**How:**
```json
// In marketplace.json, find the plugin entry
"version": "1.0.0"  // bump to "1.0.1"
```

### Skills Are Auto-Invocable (NEW)
As of Claude Code 2.1, skills are visible in the slash command menu by default and Claude can auto-invoke them based on description. No wrapper commands needed.

To opt-out of slash menu visibility:
```yaml
---
name: skill-name
user-invocable: false  # Hide from slash menu
---
```

### No plugin.json Needed
Individual plugins do NOT need `.claude-plugin/plugin.json`. The marketplace.json handles all metadata. Auto-discovery finds commands/ and skills/ directories.

### Single README Policy
All documentation in root README.md. No per-skill READMEs. Prevents drift.

### Testing Changes
Always test in a SEPARATE project after pushing:
```bash
/plugin marketplace update claude-hacks
/plugin install {plugin-name}@claude-hacks
```
</critical_learnings>

<skill_authoring>
## Skill File Format

```markdown
---

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mahidalhan/claude-hacks](https://github.com/mahidalhan/claude-hacks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
