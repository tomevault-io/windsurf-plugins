---
trigger: always_on
description: OPC Skills is a library of reusable AI agent skills for solopreneurs, indie hackers, and one-person companies. Each skill extends AI agents with specialized capabilities through structured instructions and automation scripts.
---

# OPC Skills

OPC Skills is a library of reusable AI agent skills for solopreneurs, indie hackers, and one-person companies. Each skill extends AI agents with specialized capabilities through structured instructions and automation scripts.

## Core Commands

- **Lint**: `npm run lint` (website) / `python3 -m pylint skills/*/scripts/*.py` (skills)
- **Type-check**: `npm run typecheck` (website)
- **Test**: `npm run test` (website)
- **Dev**: `npm run dev` (website)
- **Build**: `npm run build` (website)
- **Publish**: See [Release Process](#release-process)

## Project Layout

```
├── skills/                    → Source skill implementations
│   ├── requesthunt/           → RequestHunt skill
│   ├── domain-hunter/         → Domain search & pricing
│   ├── logo-creator/          → Logo generation
│   ├── banner-creator/        → Banner creation
│   ├── nanobanana/            → Gemini image generation
│   ├── reddit/                → Reddit integration
│   ├── twitter/               → Twitter/X integration
│   ├── producthunt/           → Product Hunt integration
│   └── seo-geo/               → SEO & GEO optimization
├── .factory/skills/           → Installed skill versions
├── skills.json                → Global metadata and version registry
├── website/                   → Documentation website
├── CHANGELOG.md               → Version history
└── .factory/AGENTS.md         → This file
```

## Skill Structure

Each skill follows this standard structure:

```
skills/<skill-name>/
├── SKILL.md                   → Main skill documentation (required)
│   └── YAML frontmatter:
│       - name: skill identifier
│       - description: what it does
│       - triggers: activation keywords
│       - dependencies: required skills (e.g., ["twitter", "reddit"])
├── scripts/                   → Executable Python/Shell scripts
├── examples/                  → Usage examples
└── references/                → API docs and references
```

**SKILL.md** must include YAML frontmatter with these fields:
- `name`: Unique identifier (kebab-case)
- `description`: Clear description of functionality
- `triggers`: Keywords that activate the skill
- `dependencies`: List of required skill names (optional)

Example:
```yaml
---
name: domain-hunter
description: Search domains, compare registrar prices, and find promo codes
triggers:
  - domain
  - registrar
  - buy domain
dependencies:
  - twitter
  - reddit
---
```

## Skill Development Patterns

### Creating a New Skill

1. **Create skill directory**
   ```bash
   mkdir -p skills/your-skill-name/scripts
   ```

2. **Write SKILL.md** with proper YAML frontmatter and usage instructions

3. **Add Python/Shell scripts** in `scripts/` directory with clear CLI interfaces

4. **Include examples** in `examples/` directory showing real workflows

5. **Add references** in `references/` for API documentation and resources

### Skill Script Guidelines

- Use clear argument names and help text (`--help`)
- Output machine-readable formats (JSON for data, markdown for reports)
- Include error handling and meaningful error messages
- Document dependencies and environment variables required
- Use relative paths or `${SKILL_DIR}` for portability
- When a skill adds CLI flags, scripts, or pricing changes, update `SKILL.md` examples and API notes in the same change so the docs stay aligned with the shipped behavior.

### Dependencies & Composition

When a skill requires other skills:
- List them in SKILL.md `dependencies` field
- Document which skills and minimum versions are needed
- Users install dependencies via: `npx skills add ReScienceLab/opc-skills --skill <dependency>`

Example (from domain-hunter):
```yaml
dependencies:
  - twitter
  - reddit
```

## Versioning & Dependencies Strategy

### Semantic Versioning

All version numbers follow [Semantic Versioning](https://semver.org/):

```
MAJOR.MINOR.PATCH
- MAJOR: Breaking changes or API incompatibility
- MINOR: New features (backward compatible)
- PATCH: Bug fixes
```

### Individual Skill Versions

Each skill has its own version number in `skills.json`:

```json
{
  "name": "domain-hunter",
  "version": "1.0.0",
  "dependencies": ["twitter", "reddit"]
}
```

**Version updates:**
- Update version when skill functionality changes
- Update `skills.json` entry
- Add entry to `CHANGELOG.md` under the appropriate version

### Global Project Version

The root `version` in `skills.json` tracks the overall project:
- Updated during major releases coordinating multiple skills
- Useful for milestone tracking (1.0.0 = Initial release, 2.0.0 = Major refactor, etc.)
- Individual skills can increment independently

### Dependency Compatibility

When updating dependencies:
1. Test that dependent skills still work with new versions
2. Document breaking changes clearly in CHANGELOG.md
3. Consider running dependent skill tests before merging

**Example breaking change:**
```markdown
## [2.0.0] - 2025-XX-XX

### Changed
- **seo-geo**: Updated DataForSEO API integration (API v3.0 required)

### Fixed
- **twitter**: Fixed rate limiting handling (requires twitter >= 1.1.0)
```

## Git Workflow


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ReScienceLab/opc-skills](https://github.com/ReScienceLab/opc-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
