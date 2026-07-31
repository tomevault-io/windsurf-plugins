---
trigger: always_on
description: **Last Updated**: Mon Oct 27 19:39:26 CST 2025
---

# Templates Module

**Last Updated**: Mon Oct 27 19:39:26 CST 2025
[Root](../CLAUDE.md) > **templates**

## Module Responsibilities

Template configuration module providing multilingual configuration templates, workflow templates, and AI memory templates, supporting both Chinese and English environments for comprehensive Claude Code environment setup.

## Entry Points and Startup

- **Main Entry Points**:
  - `common/` - Common configuration templates
  - `zh-CN/` - Chinese template collection
  - `en/` - English template collection

## External Interfaces

### Template Structure

```
templates/
├── common/                    # Common configuration templates
│   ├── CLAUDE.md             # Base CLAUDE.md template
│   └── settings.json         # Claude settings template
├── zh-CN/                    # Chinese template collection
│   ├── memory/               # AI memory templates
│   └── workflow/             # Workflow templates
└── en/                       # English template collection
    ├── memory/               # AI memory templates
    └── workflow/             # Workflow templates
```

### Template Category System

```typescript
// Template categories
interface TemplateCategories {
  memory: {
    'mcp.md': string // MCP service guidance
    'technical-guides.md': string // Technical implementation guides
    'personality.md': string // AI personality configuration
    'rules.md': string // Core programming principles
  }
  workflow: {
    common: WorkflowCommands // Common tools workflow
    plan: WorkflowCommands // Feature planning workflow
    sixStep: WorkflowCommands // Six-step development workflow
    bmad: WorkflowCommands // BMad enterprise workflow
    git: WorkflowCommands // Git operation workflow
  }
}
```

## Key Dependencies and Configuration

### Template Installation Process

- Templates are copied to `~/.claude/` directory during initialization
- Language-specific templates selected based on user configuration
- Automatic template merging and update mechanisms
- Template validation and consistency checking

### Configuration Files

- `common/settings.json` - Base Claude settings template
- `common/CLAUDE.md` - Project guidance template
- Language-specific memory and workflow templates

## Data Models

### Workflow Template Structure

#### Common Tools Workflow (`common/`)

- **Commands**: `init-project.md` - Project initialization command
- **Agents**: `init-architect.md`, `get-current-datetime.md` - Core utility agents

#### Feature Planning Workflow (`plan/`)

- **Commands**: `feat.md` - Feature development command
- **Agents**: `planner.md`, `ui-ux-designer.md` - Planning and design agents

#### Six-Step Development Workflow (`sixStep/`)

- **Commands**: `workflow.md` - Six-step development process
- **Agents**: None - Process-oriented workflow

#### BMad Enterprise Workflow (`bmad/`)

- **Commands**: `bmad-init.md` - BMad initialization
- **Agents**: Complete enterprise development team simulation

#### Git Workflow (`git/`)

- **Commands**: `git-commit.md`, `git-worktree.md`, `git-cleanBranches.md`, `git-rollback.md`
- **Agents**: None - Git operation commands

### Memory Template Structure

#### AI Memory Templates

- **mcp.md**: MCP service usage guidelines and best practices
- **technical-guides.md**: Technical execution guidelines and standards
- **personality.md**: AI assistant behavior and personality configuration
- **rules.md**: Core programming principles and workflow methodology

### Template Language Support

#### Chinese Templates (`zh-CN/`)

- Complete Chinese localization for all templates
- Chinese AI interaction patterns
- Chinese technical documentation standards
- Chinese workflow descriptions

#### English Templates (`en/`)

- Complete English localization for all templates
- English AI interaction patterns
- English technical documentation standards
- English workflow descriptions

## Testing and Quality

### Template Validation Strategy

- **Consistency Testing**: Verify Chinese-English template correspondence
- **Syntax Testing**: Validate markdown syntax and formatting
- **Content Testing**: Verify template completeness and accuracy
- **Integration Testing**: Test template installation and configuration

### Quality Metrics

#### ✅ Template Completeness

- Chinese-English template parity: **100%**
- Workflow category coverage: **5/5 categories**
- Memory template coverage: **4/4 templates**
- Command template coverage: **Complete**

#### ✅ Language Support

- Chinese localization: **Complete**
- English localization: **Complete**
- Template structure consistency: **Validated**
- Content accuracy: **Verified**

#### ✅ Installation Testing

- Template copying mechanism: **Tested**
- Language selection logic: **Verified**
- Template merging: **Functional**
- Update mechanism: **Working**

### Test Coverage

- **Template Tests**: `tests/templates/chinese-templates.test.ts`
- **Installation Tests**: Integration tests for template deployment
- **Validation Tests**: Template structure and content verification

## FAQ

### Q: How to add new workflow templates?

1. Add template files in both `zh-CN/workflow/` and `en/workflow/`
2. Update workflow configuration in `src/config/workflows.ts`
3. Add corresponding translation keys in i18n files

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [miounet11/ccjk](https://github.com/miounet11/ccjk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
