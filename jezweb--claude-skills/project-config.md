---
trigger: always_on
description: **Repository**: https://github.com/jezweb/claude-skills
---

# Claude Skills

**Repository**: https://github.com/jezweb/claude-skills
**Owner**: Jeremy Dawes (Jez) | Jezweb

Production workflow skills for Claude Code CLI. Each skill guides Claude through a recipe to produce tangible output — not knowledge dumps, but working deliverables.

## Philosophy

- Every skill must produce visible output (files, configurations, deployable projects)
- "The context window is a public good" — only include what Claude doesn't already know
- **Teach patterns, not ship scripts** — skills teach Claude *what* to do, Claude generates scripts adapted to the user's environment. Pre-built scripts in `scripts/` are the rare exception, not the default. Put proven implementation patterns in `references/` for Claude to adapt.
- Follows the official Claude Code plugin spec

## Directory Structure

```
claude-skills/
├── plugins/                                # 10 plugins, 63 skills
│   ├── cloudflare/                         # Cloudflare Workers, Hono, D1/Drizzle, Vite, TanStack Start
│   │   └── skills/
│   │       ├── cloudflare-worker-builder/
│   │       ├── vite-flare-starter/
│   │       ├── tanstack-start/
│   │       ├── hono-api-scaffolder/
│   │       ├── d1-drizzle-schema/
│   │       ├── d1-migration/
│   │       ├── db-seed/
│   │       └── cloudflare-api/
│   ├── web-design/                         # Local business SEO
│   │   └── skills/
│   │       └── seo-local-business/
│   ├── frontend/                           # Tailwind v4 + shadcn/ui + landing pages + showcases + React + design
│   │   └── skills/
│   │       ├── tailwind-theme-builder/
│   │       ├── shadcn-ui/
│   │       ├── landing-page/
│   │       ├── product-showcase/
│   │       ├── react-patterns/
│   │       ├── design-review/
│   │       ├── react-native/
│   │       ├── design-loop/
│   │       ├── design-system/
│   │       └── walkthrough-video/
│   ├── design-assets/                      # Colour palettes, favicons, icons, image processing, AI images
│   │   └── skills/
│   │       ├── color-palette/
│   │       ├── favicon-gen/
│   │       ├── icon-set-generator/
│   │       ├── image-processing/
│   │       └── ai-image-generator/
│   ├── integrations/                       # Google Workspace, ElevenLabs, MCP, NemoClaw
│   │   └── skills/
│   │       ├── gws-setup/
│   │       ├── gws-install/
│   │       ├── google-chat-messages/
│   │       ├── google-apps-script/
│   │       ├── elevenlabs-agents/
│   │       ├── mcp-builder/
│   │       ├── nemoclaw-setup/
│   │       ├── parcel-tracking/
│   │       └── stripe-payments/
│   ├── dev-tools/                          # Context, sessions, releases, brains trust, git, browser automation
│   │   └── skills/
│   │       ├── project-health/
│   │       ├── project-docs/
│   │       ├── app-docs/
│   │       ├── github-release/
│   │       ├── brains-trust/
│   │       ├── git-workflow/
│   │       ├── team-update/
│   │       ├── ux-audit/
│   │       ├── responsiveness-check/
│   │       ├── agent-browser/
│   │       ├── deep-research/
│   │       ├── onboarding-ux/
│   │       ├── fork-discipline/
│   │       ├── roadmap/
│   │       └── vitest/
│   ├── shopify/                            # Shopify store management
│   │   └── skills/
│   │       ├── shopify-setup/
│   │       ├── shopify-products/
│   │       └── shopify-content/
│   ├── wordpress/                          # WordPress content & Elementor
│   │   └── skills/
│   │       ├── wordpress-setup/
│   │       ├── wordpress-content/
│   │       └── wordpress-elementor/
│   ├── social-media/                       # Social media content creation
│   │   └── skills/
│   │       └── social-media-posts/
│   └── writing/                            # Business English + professional documents
│       └── skills/
│           ├── aussie-business-english/
│           ├── us-business-english/
│           ├── uk-business-english/
│           ├── nz-business-english/
│           ├── resume-cover-letter/
│           ├── proposal-writer/
│           ├── award-application/
│           └── strategy-document/
├── .claude-plugin/                         # Marketplace + plugin config
│   ├── marketplace.json
│   └── plugin.json
├── CLAUDE.md                               # This file
├── README.md                               # Public-facing overview
└── LICENSE                                 # MIT
```

## Plugin Anatomy (Anthropic Spec)

Each plugin contains one or more skills, auto-discovered from `skills/`:

```
plugin-name/
├── .claude-plugin/
│   └── plugin.json        # name, description, author
└── skills/
    └── skill-name/
        ├── SKILL.md       # Frontmatter + instructions (inline everything critical)
        ├── ERRATA.md      # Optional: versioned corrections discovered during builds
        ├── scripts/       # Executable scripts the agent RUNS (not reads)
        ├── references/    # Supplementary/variant docs (NOT critical path)
        └── assets/        # Files used in output (templates, images)
```

## Adding a New Plugin

1. Create the plugin directory:
   ```bash
   mkdir -p plugins/my-plugin/{.claude-plugin,skills}
   ```

2. Create `.claude-plugin/plugin.json`:
   ```json
   {
     "name": "my-plugin",
     "description": "What this plugin does.",

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jezweb/claude-skills](https://github.com/jezweb/claude-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
