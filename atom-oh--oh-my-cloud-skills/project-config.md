---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A Claude Code plugin marketplace containing six plugins for AWS cloud work:
- **aws-content-plugin** — Content creation (presentations, diagrams, docs, workshops)
- **aws-ops-plugin** — Infrastructure operations & troubleshooting (EKS, networking, IAM, observability)
- **kiro-power-converter** — Convert Claude Code plugins to Kiro IDE Power format
- **agentcore-creator** — Convert Claude Code plugins to Bedrock AgentCore
- **kiro-review** — Architecture deep review via Kiro CLI
- **project-init** — Project scaffolding and documentation management

All plugins are installed via `/plugin marketplace add` or loaded locally with `--plugin-dir`.

## Development Commands

```bash
# Load plugins locally for testing
claude --plugin-dir ./plugins/aws-content-plugin
claude --plugin-dir ./plugins/aws-ops-plugin

# Validate plugin manifests
python3 -c "import json; d=json.load(open('plugins/aws-content-plugin/.claude-plugin/plugin.json')); print(f'content: {len(d[\"agents\"])} agents, {len(d[\"skills\"])} skills')"
python3 -c "import json; d=json.load(open('plugins/aws-ops-plugin/.claude-plugin/plugin.json')); print(f'ops: {len(d[\"agents\"])} agents, {len(d[\"skills\"])} skills')"

# Verify all plugin.json references resolve to existing files
cd plugins/aws-ops-plugin && python3 -c "
import json, os
d = json.load(open('.claude-plugin/plugin.json'))
for a in d['agents']:
    assert os.path.isfile(a.lstrip('./')), f'Missing agent: {a}'
for s in d['skills']:
    assert os.path.isfile(s.lstrip('./') + '/SKILL.md'), f'Missing skill: {s}'
print('All references OK')
"

# Remarp VSCode Extension development
cd tools/remarp-vscode
npm install && npm run compile    # Build TypeScript
npx vsce package                  # Package .vsix
code --install-extension remarp-vscode-0.1.0.vsix  # Install locally

# Evaluate skills (quality, structure, token usage)
python3 scripts/eval-skills.py
python3 scripts/eval-skills.py --plugin aws-content-plugin --skill reactive-presentation

# Behavioral eval (E2E skill runtime testing via claude --print)
python3 scripts/eval-skill-behavior.py --skill reactive-presentation --dry-run
python3 scripts/eval-skill-behavior.py --case evals/reactive-presentation/flow-layout.yaml
python3 scripts/eval-skill-behavior.py --skill reactive-presentation --ci --threshold 70

# Validate Remarp source (rejection loop — run before build)
python3 plugins/aws-content-plugin/skills/reactive-presentation/scripts/remarp_to_slides.py validate <project-dir>/
python3 plugins/aws-content-plugin/skills/reactive-presentation/scripts/remarp_to_slides.py validate <project-dir>/ --json
```

## Plugin Architecture

Each plugin follows the same structure:

```
plugins/<plugin-name>/
├── .claude-plugin/plugin.json    # Manifest: agents[], skills[], mcpServers{}
├── CLAUDE.md                     # Auto-invocation keyword → agent routing rules
├── agents/<name>.md              # Agent definitions (YAML frontmatter + markdown body)
└── skills/<name>/                # Skill directories
    ├── SKILL.md                  # Entry point (YAML frontmatter with triggers)
    ├── references/               # Distilled knowledge docs
    └── templates/                # Templates (content-plugin only)
```

### Agent File Format

Every agent `.md` file has YAML frontmatter with exactly three fields:

```yaml
---
name: eks-agent
description: "Description with trigger keywords."
tools: Read, Write, Glob, Grep, Bash, AskUserQuestion
---
```

The body contains: Core Capabilities, Diagnostic Commands, Decision Tree (Mermaid), Error→Solution mapping, MCP Integration, Reference Files, Output Format.

### Skill File Format

Each `SKILL.md` has frontmatter with `name`, `description`, and `triggers` (keyword list). The `references/` subdirectory holds distilled operational knowledge extracted from source docs.

### MCP Configuration

`aws-ops-plugin` bundles 2 MCP servers in `plugin.json` → `mcpServers`:
- `awsdocs` (stdio/uvx) — Official AWS documentation search
- `awsapi` (stdio/uvx) — Direct AWS API calls

The remaining 3 servers are provided by the `deploy-on-aws` plugin (available when both plugins are loaded):
- `awsknowledge` (HTTP) — Architecture recommendations
- `awspricing` (stdio/uvx) — Pricing data
- `awsiac` (stdio/uvx) — CloudFormation/CDK validation

### Hooks

Plugins use `hooks` in `plugin.json` for automated checks:
- **PostToolUse (Bash)** — Detects build warnings in `remarp_to_slides.py` output (content), AWS error patterns (ops)
- **PostToolUse (Edit/Write)** — Detects reactive-presentation skill file changes, validates Remarp frontmatter and slide notes
- **SessionStart** — Plugin load announcements with domain context

### Auto-Invocation

Each plugin's `CLAUDE.md` defines keyword→agent routing tables. Keywords include both English and Korean terms. When a user prompt matches keywords, the corresponding agent activates automatically.

## Versioning

All plugins share a single version tracked in their `plugin.json` → `"version"` field, mirrored in `marketplace.json`. Git tags **must** match this version.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Atom-oh/oh-my-cloud-skills](https://github.com/Atom-oh/oh-my-cloud-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
