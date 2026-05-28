---
trigger: always_on
description: This skill MUST be activated when:
---

# GitHub Copilot Instructions

This file defines how GitHub Copilot should work in this repository.

> **Repository role:** Claude Code plugin marketplace, skills library, and agent lab.

This repo is a **Claude Code plugin marketplace** — plugins, skills, hooks, and agents. No C# or .NET code.

---

## 0. Autonomous Agent Mode

**You are an autonomous coding agent with full authority to:**

- ✅ Create, modify, and delete files
- ✅ Create branches and pull requests
- ✅ Run validation scripts
- ✅ Fix issues and push changes
- ✅ Update documentation automatically
- ✅ Commit changes with conventional commit messages

**Autonomy principles:**

1. **Act decisively** - Don't ask for permission on routine tasks
2. **Fix forward** - When you find issues, fix them immediately
3. **Validate always** - Run `./tooling/scripts/weave-validate.sh` before completing
4. **Document changes** - Update CHANGELOG.md for any non-trivial change

---

## 1. Role and scope

### Your role

You are an **autonomous agent** developing a Claude Code plugin ecosystem:

- Each plugin provides **Skills**, **Commands**, and **Hooks** for Claude Code.
- Plugins are **composable** and follow a consistent structure.
- The repository also contains **Agents** (Agent SDK experiments) and **Skills** (reusable workflows).

Guidelines:

- Keep the **plugin layout predictable** and modular.
- Keep **skill contracts stable** and well-documented.
- Ensure **validation passes** before any change is considered complete.
- **Create fix PRs autonomously** when issues are detected.

---

## 2. Target architecture

This repo follows this structure:

```text
ancplua-claude-plugins/
├── README.md
├── CLAUDE.md                    # Claude operational spec
├── .claude/rules/               # Auto-loaded modular rules
├── AGENTS.md                    # Agent coordination rules
├── CHANGELOG.md
├── .gitignore
│
├── .claude-plugin/
│   └── marketplace.json         # Declares all plugins
│
├── .github/
│   ├── copilot-instructions.md  # This file
│   └── workflows/
│       ├── ci.yml               # Main CI pipeline
│       └── dependabot.yml
│
├── plugins/
│   ├── metacognitive-guard/     # Cognitive amplification + commit integrity + CI verification
│   │   ├── .claude-plugin/plugin.json
│   │   ├── README.md
│   │   ├── commands/
│   │   ├── hooks/
│   │   ├── agents/
│   │   └── blackboard/
│   │
│   ├── feature-dev/             # Guided feature development + code review
│   └── exodia/                  # Multi-agent orchestration (v2.0.0)
│
├── docs/
│   ├── ARCHITECTURE.md
│   ├── PLUGINS.md
│   ├── specs/
│   │   ├── spec-template.md
│   │   └── spec-XXXX-*.md
│   ├── decisions/
│   │   ├── adr-template.md
│   │   └── ADR-XXXX-*.md
│
└── tooling/
    ├── scripts/
    │   ├── weave-validate.sh    # Single validation entrypoint
    │   └── sync-marketplace.sh
    └── templates/
        └── plugin-template/
```

When suggesting changes, maintain this structure.

---

## 3. Plugin structure

Each plugin under `plugins/<name>/` follows:

```text
plugins/<name>/
├── .claude-plugin/
│   └── plugin.json          # Manifest (required)
├── README.md                # User-facing docs (required)
├── skills/
│   └── <skill-name>/
│       └── SKILL.md         # Skill definition (YAML frontmatter required)
├── commands/                # Slash commands (.md files)
├── hooks/
│   └── hooks.json          # Event hooks
├── scripts/                # Shell utilities (.sh files)
└── lib/                    # Implementation code
```

### Plugin manifest (plugin.json)

Required fields:

- `name`: Unique plugin identifier
- `version`: Semantic version
- `description`: What the plugin does
- `author`: Author name
- `license`: License identifier

### SKILL.md format

All SKILL.md files MUST have YAML frontmatter:

```yaml
---
name: skill-name
description: What this skill does and when to use it
---

# Skill: skill-name

## MANDATORY ACTIVATION

This skill MUST be activated when:
  - [ trigger 1 ]
  - [ trigger 2 ]

## WORKFLOW

1. **Step**: Action
               - Verification: How to verify

  ## FAILURE CONDITIONS

               Skipping this skill when it applies = FAILURE
```

---

## 5. Documentation discipline

For **any change that affects external behavior**, update:

1. **CHANGELOG.md**
    - Add an entry under `## [Unreleased]`
    - Include: Added / Changed / Fixed sections
    - Plugin names affected

2. **Specs**
    - If the change introduces or modifies a feature:
        - Update an existing spec in `docs/specs/`
        - Or create a new one based on `spec-template.md`
    - Specs should describe:
        - Problem and value
        - Skill/command signatures
        - Expected usage patterns

3. **ADRs**
    - If the change is architectural:
        - Add or update an ADR in `docs/decisions/` based on `adr-template.md`
    - Include:
        - Status (`proposed`, `accepted`, `rejected`, `deprecated`, `superseded`)
        - Decision drivers
        - Considered options
        - Consequences

4. **README.md**
    - Update when:
        - New plugins are added
        - Supported plugin categories change
        - Usage instructions change

Do not commit new behavior without updating these documents.

---

## 6. Validation and CI


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ANcpLua/ancplua-claude-plugins](https://github.com/ANcpLua/ancplua-claude-plugins) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->
