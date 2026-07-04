---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repository Is

This is a **skills source repository** — one of potentially many that feed into the [agentic-catalog](https://github.com/RHEcosystemAppEng/agentic-catalog).  It contains agentic packs with skills, MCP server configurations, AI-optimized documentation, and catalog metadata for Red Hat platforms.

Contributors work here to create, improve, and validate skills. An internal process periodically fetches content from this repository (and others like it) to build the unified catalog and marketplace. **This repo does not serve the marketplace directly** — it is a source of skills that the catalog aggregates.

## Repository Structure

```
agentic-plugins/
├── rh-sre/              # Site Reliability Engineering pack (reference implementation)
├── rh-developer/        # Developer tools pack
├── ocp-admin/           # OpenShift administration pack
├── rh-virt/             # Virtualization management pack
├── rh-basic/            # Getting started pack
├── rh-ai-engineer/      # AI Engineering pack
├── rh-automation/       # Automation pack
├── rh-support-engineer/ # Support engineering pack
├── eval/                # Skill evaluation reports (report.json + report.md per skill)
├── scripts/             # Validation and CI helper scripts
├── catalog/             # JSON Schema for .catalog/collection.yaml validation
│   └── schema.yaml
└── .claude/skills/      # Repo-level Claude Code skills (contribution, linting, compliance)
```

### `catalog/schema.yaml`

This file defines the JSON Schema used by `validate_collection_schema.py` and `validate_collection_compliance.py` to validate each pack's `.catalog/collection.yaml`. It is not related to the catalog marketplace repository — it is a validation artifact that ensures catalog metadata is well-formed before the catalog build process consumes it.

### Agentic Pack Architecture

Each pack is persona-specific and follows this structure:

```
<pack-name>/
├── AGENTS.md            # AI Context Module instruction routing (persona, skills, rules)
├── README.md            # Pack description, persona, target marketplaces
├── mcps.json            # MCP server configurations (uses env vars for credentials)
├── .catalog/            # Collection metadata consumed by the catalog build process
│   ├── collection.yaml  # Pack catalog definition (golden source for catalog)
│   └── collection.json  # Deterministic JSON mirror of collection.yaml
├── skills/              # Specialized task executors (including orchestration skills)
│   └── <skill>/
│       └── SKILL.md     # Skill definition with YAML frontmatter
└── docs/                # AI-optimized knowledge base (optional, rh-sre reference)
```

### Relationship with the Catalog

Each pack's `.catalog/` directory contains metadata that describes the pack for the marketplace. This metadata stays here, alongside the skills it describes. The catalog build process reads it from this repo to assemble the unified marketplace. The golden sources are always `SKILL.md`, `AGENTS.md`, `README.md`, and `mcps.json` — `.catalog/` is derived from them, never the other way around.

## Contributing

Skills are added directly to this repository, inside an existing pack. The contributor opens a PR, skills are reviewed and merged, and maintainers own them from that point. Use `/agentic-contribution-skill` in Claude Code or follow [CONTRIBUTING.md](CONTRIBUTING.md).

## Working with Skills

**Skills** (`skills/<skill-name>/SKILL.md`):
- Single-purpose task executors
- Encapsulate specific tool access and domain knowledge
- Invoked via the `Skill` tool
- Structure: YAML frontmatter + implementation guide

**Key Pattern**: Skills encapsulate tools; orchestration skills invoke other skills. Never call MCP tools directly — always go through skills.

## Skill and Agent Requirements

**CRITICAL:** EVERY SKILL and AGENT must comply with:
- **Tier 1:** agentskills.io specification (AUTOMATED via linter)
- **Tier 2:** Repository design principles (MANUAL review)

The catalog's internal process applies its own evaluation and assigns a scorecard, but skills must pass Tier 1 and Tier 2 here before merging.

**Before committing any skill:**

1. **Run automated validation (Tier 1):**
   ```bash
   uv run python scripts/validate_skills_tier1.py <pack>/skills/<skill-name>/SKILL.md
   ```

2. **Manual review (Tier 2):**
   - Review [SKILL_DESIGN_PRINCIPLES.md](SKILL_DESIGN_PRINCIPLES.md) for complete requirements
   - Use appropriate template (general or collection-specific)

3. **Full validation:**
   ```bash
   make validate
   ```

**Documentation:**
- [SKILL_DESIGN_PRINCIPLES.md](SKILL_DESIGN_PRINCIPLES.md) - Complete design principles, templates, and rationale

### MCP Server Integration

MCP servers are configured in `<pack>/mcps.json`:
```json
{
  "mcpServers": {
    "server-name": {
      "command": "podman|docker|npx",
      "args": ["..."],
      "env": {
        "VAR_NAME": "${VAR_NAME}"
      },
      "security": {
        "isolation": "container",
        "network": "local",
        "credentials": "env-only"
      }
    }
  }
}
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RHEcosystemAppEng/agentic-plugins](https://github.com/RHEcosystemAppEng/agentic-plugins) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
