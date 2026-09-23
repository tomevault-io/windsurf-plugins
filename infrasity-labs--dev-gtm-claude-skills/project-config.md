---
trigger: always_on
description: This repo is a library of **Claude Code skills, agents, and extensions** — reusable AI-powered workflows for SEO, marketing, content, web design, and product management. Skills are invoked as slash commands inside Claude Code (e.g. `/seo-audit`, `/blog-write`, `/analytics`).
---

# AGENTS.md — Claude Code Skills Library

This repo is a library of **Claude Code skills, agents, and extensions** — reusable AI-powered workflows for SEO, marketing, content, web design, and product management. Skills are invoked as slash commands inside Claude Code (e.g. `/seo-audit`, `/blog-write`, `/analytics`).

---

## Repo Structure

```
dev-gtm-claude-skills/
├── seo-skills/                  # SEO cluster (~35 skills + 8 extensions)
│   ├── seo/                     # Master SEO orchestrator
│   ├── seo-audit/               # Full site audit
│   ├── seo-cluster/             # Topic clustering
│   ├── ...                      # 30+ more SEO skills
│   └── extensions/              # MCP-backed extension plugins
│       ├── dataforseo/          # DataForSEO MCP (install.sh)
│       ├── ahrefs/              # Ahrefs MCP (install.sh)
│       ├── firecrawl/           # Firecrawl MCP (install.sh)
│       ├── bing-webmaster/      # Bing Webmaster MCP (install.sh)
│       ├── profound/            # Profound MCP (install.sh)
│       ├── seranking/           # SERanking MCP (install.sh)
│       ├── unlighthouse/        # Unlighthouse MCP (install.sh)
│       └── banana/              # Image generation MCP (install.sh)
│
├── writing-skills/              # Blog & content cluster (~35 skills)
│   ├── blog/                    # Master blog orchestrator
│   ├── blog-write/              # Full blog post writer
│   ├── blog-audit/              # Quality audit
│   └── ...
│
├── marketing-skills/            # Marketing cluster (~27 skills)
│   ├── analytics/               # GA4 / tracking setup
│   ├── ads/                     # Paid ads strategy
│   ├── competitor-profiling/    # Competitor research
│   ├── revops/                  # Revenue operations
│   └── ...
│
├── web-design/                  # Web design cluster (~4 skills)
│   ├── frontend-design/
│   ├── landing-page-auditor/
│   ├── site-architecture/
│   └── web-design-guidelines/
│
├── product-management-skills/   # PM cluster (~8 skills)
│   ├── product-manager-toolkit/ # Full PM toolkit (RICE, PRDs, interviews)
│   ├── agile-product-owner/     # Agile PO workflows
│   ├── prd-development/         # PRD creation
│   └── ...
│
├── skills/                      # Utility skills (~10 skills)
│   ├── docs-auditor/
│   ├── growth-report/
│   ├── orphan-pages-internal-linking-opportunities/
│   └── ...
│
├── .claude/                     # Claude Code runtime config (do not edit manually)
│   ├── settings.json            # MCP server config + permissions
│   ├── skills/                  # Installed skill copies (managed by install scripts)
│   ├── agents/                  # Installed agent copies (managed by install scripts)
│   └── extensions/              # Installed extension copies (managed by install scripts)
│
└── agents/                      # Source agent definitions
    ├── blog-researcher.md
    ├── blog-writer.md
    └── ...
```

---

## How Skills Work

Each skill lives in its own directory with a single `SKILL.md` file.

### SKILL.md Format

```markdown
---
name: skill-name
description: >
  One-paragraph description used for trigger matching. This is what Claude
  reads to decide when to invoke the skill. Be specific about trigger phrases.
user-invokable: true
argument-hint: "[url] [optional-flags]"
metadata:
  category: seo | blog | marketing | web-design | pm | utility
---

# Skill Title

Skill body — instructions, frameworks, prompts, and workflow steps.
```

**Key frontmatter fields:**
- `name` — must match the directory name exactly
- `description` — trigger-matching text; include all phrases that should activate this skill
- `user-invokable` — `true` means users can call it via `/skill-name`
- `argument-hint` — shown in autocomplete to guide input format
- `metadata.category` — used for grouping and discovery

---

## How Extensions Work

Extensions add MCP server integrations on top of base skills. Each extension ships with:

```
extensions/<name>/
├── install.sh       # Installs skill, agent, and MCP config into ~/.claude/
├── uninstall.sh     # Removes all installed files and MCP config
├── install.ps1      # Windows equivalent
├── uninstall.ps1
├── README.md        # Setup guide + usage examples
├── agents/          # Agent definition(s) for this extension
├── skills/          # Skill(s) this extension provides
└── docs/            # Setup documentation
```

### Installing an extension

```bash
# From the repository root
./seo-skills/extensions/dataforseo/install.sh    # prompts for API credentials
./seo-skills/extensions/ahrefs/install.sh
./seo-skills/extensions/firecrawl/install.sh
```

The install script:
1. Copies skill files into `~/.claude/skills/`
2. Copies agent files into `~/.claude/agents/`
3. Writes MCP server config into `~/.claude/settings.json`
4. Pre-warms the npm package

### Uninstalling

```bash
./seo-skills/extensions/dataforseo/uninstall.sh
```

---

## Adding a New Skill

1. Create a directory under the appropriate cluster:
   ```bash
   mkdir seo-skills/my-new-skill
   ```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Infrasity-Labs/dev-gtm-claude-skills](https://github.com/Infrasity-Labs/dev-gtm-claude-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
