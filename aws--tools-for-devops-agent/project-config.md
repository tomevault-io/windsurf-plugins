---
trigger: always_on
description: This skill [what it does] by [how it does it] to [outcome].
---

# Project Conventions

This repository consolidates open-source tools for AWS DevOps Agent — skills, custom agents, and MCP servers, plus supporting infrastructure templates. Follow these conventions when contributing. See [CONTRIBUTING.md](../CONTRIBUTING.md) for the full contribution workflow.

## Key References

- [Agent Skills spec](https://agentskills.io/home) — the open standard this project follows for skill structure
- [AWS DevOps Agent Skills documentation](https://docs.aws.amazon.com/devopsagent/latest/userguide/about-aws-devops-agent-devops-agent-skills.html) — official AWS docs on creating and uploading skills
- [AWS DevOps Agent custom agents documentation](https://docs.aws.amazon.com/devopsagent/latest/userguide/working-with-devops-agent-custom-agents-index.html) — official AWS docs on custom agents
- [AGENTS.md specification](https://agents.md/) — the open standard for custom agent definitions
- [Model Context Protocol](https://modelcontextprotocol.io) — the open standard for MCP servers
- [Connecting MCP servers to DevOps Agent](https://docs.aws.amazon.com/devopsagent/latest/userguide/configuring-integrations-and-knowledge-connecting-mcp-servers.html) — official AWS docs on registering MCP servers

## Repository Structure

```
tools-for-devops-agent/
├── README.md                 # Project overview with skills/agents/MCP tables
├── CONTRIBUTING.md           # Contribution guidelines
├── llms.txt                  # Structured repo overview for AI tools
├── .gitignore                # Root-level ignores
├── cloudformation/
│   └── devops-agent-skill-policies.yaml  # IAM policies skills require
├── docs/                     # GitHub Pages (mkdocs) documentation site
├── skills/
│   ├── .gitignore            # Allowlist for DevOps Agent supported extensions only
│   └── <skill-name>/
│       ├── SKILL.md          # Required: main skill instructions with frontmatter
│       ├── README.md         # Skill documentation (purpose, prompts, upload instructions)
│       ├── CHANGELOG.md      # Version history
│       ├── evals/            # Required: skill evaluation tool output (generated)
│       │   ├── evals.json
│       │   ├── structure/    # structure-tests-results-v<N>.json
│       │   ├── best-practices/  # v<N>/benchmark.json, v<N>/iteration-<n>/
│       │   └── functional/   # v<N>/benchmark.json, v<N>/iteration-<n>/
│       ├── assets/           # Optional: images, diagrams, data files
│       └── references/       # Optional: supplementary reference docs
├── custom-agents/
│   └── <agent-name>/
│       ├── SYSTEM_PROMPT.md  # Required: the agent's system prompt
│       ├── README.md         # Agent documentation
│       └── CHANGELOG.md      # Version history
└── mcp/
    └── <server-name>/
        ├── README.md         # Server documentation and deployment steps
        └── ...               # Server implementation and deployment assets
```

## Writing Skills

Skills live under `skills/` and should follow both the [Agent Skills spec](https://agentskills.io/home) best practices and [AWS DevOps Agent best practices](https://docs.aws.amazon.com/devopsagent/latest/userguide/about-aws-devops-agent-devops-agent-skills.html). Skills are the most common contribution type; the guidance below is the most detailed for that reason.

### SKILL.md Requirements

- Must include valid frontmatter with `name` and `description` fields.
- `name`: lowercase letters, numbers, and hyphens only (max 64 characters, no leading/trailing hyphens).
- `description`: written from the agent's perspective, specifying when and why the skill should activate. Be specific about scenarios, services, error types, or symptoms that should trigger the skill. Minimum 100 characters recommended.
- Instructions should be step-by-step, actionable, and include decision trees for different scenarios.
- Include expected outputs and success criteria.
- Reference specific AWS APIs, CLI commands, or tools the agent should use.
- Use tables for structured data (e.g., filtering strategies, relevance scoring).

### SKILL.md Frontmatter Example

```yaml
---
name: my-skill-name
description: Use this skill when investigating [specific scenarios].
  Activate when you observe [specific symptoms, error patterns, or conditions].
  This skill [what it does] by [how it does it] to [outcome].
metadata:
  author: github-username
  version: "1.0.0"
---
```

The `metadata` block with `author` and `version` fields is required. Initial version should be `"1.0.0"`.

### Skill README.md Structure

Each skill must have a README.md following this structure (see `skills/support-cases/README.md` as reference):

1. **Title** — skill name as heading
2. **Purpose** — what the skill does and why it's useful
3. **Key Capabilities** — bullet list of what the skill enables
4. **Prerequisites** — what's needed before using the skill (IAM permissions, service plans, etc.)
5. **Limitations** — known constraints or boundaries
6. **Agent Types** — which DevOps Agent types use this skill
7. **Uploading to AWS DevOps Agent** — zip command and upload steps
8. **How to Use This Skill** — sample prompts organized by agent type/use-case

### Changelog

Every skill must include a `CHANGELOG.md` tracking version history. Use semantic versioning:

```markdown
# Changelog


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aws/tools-for-devops-agent](https://github.com/aws/tools-for-devops-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
