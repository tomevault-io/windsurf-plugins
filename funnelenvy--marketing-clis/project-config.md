---
trigger: always_on
description: This is a meta-repository that orchestrates the creation and maintenance of open source CLI tools for marketing platforms that lack them. Each marketing tool gets its own **standalone GitHub repo** for discoverability (easier to find via search), with cross-references back to this meta-repo. This repo contains:
---

# Marketing CLIs — Meta Repository

## Project Vision

This is a meta-repository that orchestrates the creation and maintenance of open source CLI tools for marketing platforms that lack them. Each marketing tool gets its own **standalone GitHub repo** for discoverability (easier to find via search), with cross-references back to this meta-repo. This repo contains:

1. **Registry** — index of all generated CLIs with status, links, and metadata
2. **Generator skill** — `/generate-cli` Claude Code skill that creates new CLI repos from API documentation
3. **Shared libraries** — common auth, output formatting, config, and rate limiting patterns
4. **Templates** — repo scaffolding, CI/CD, docs, and release automation

All CLIs are Node.js (TypeScript) with single-binary distribution. Everything is MIT licensed.

## Agent Teams Strategy

This project is designed for Claude Code agent teams. The architecture naturally avoids file conflicts because each CLI is a separate repo in a sibling directory.

**Recommended workflow:**
1. **Lead agent** builds the meta-repo foundation (shared packages, template, generator skill) — this is sequential and must complete first
2. **Lead agent spawns 5 teammates**, one per CLI — each teammate owns an entire standalone repo directory and works independently
3. **Lead agent** monitors progress, synthesizes results, updates the meta-repo registry and README when teammates finish

**Why this works well for agent teams:**
- Zero file conflict risk — each teammate writes to its own `clis/{tool}-cli/` directory
- Each CLI is fully independent — no cross-teammate dependencies after shared packages are built
- Teammates can research APIs in parallel (the slowest part of CLI generation)
- Lead only needs to update `registry.json` and `README.md` in the meta-repo at the end

**Teammate spawn guidelines:**
- Each teammate prompt must include: tool name, API docs URL, auth method, priority endpoints, and the full path to create the repo
- Each teammate prompt must reference this CLAUDE.md for standards
- Teammates should be told NOT to modify anything in the meta-repo — only the lead does that
- Teammates should be told to use the shared packages from `shared/` as reference for patterns but to bundle/copy the code into their own repo

## Architecture

```
marketing_clis/                  # This meta-repo (GitHub: FunnelEnvy/marketing-clis)
├── CLAUDE.md                    # Claude Code instructions (this file)
├── README.md                    # Public-facing index — explains project, lists all CLIs
├── LICENSE                      # MIT
├── .envrc                       # CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1
├── registry.json                # Machine-readable registry of all CLIs
├── .claude/
│   └── skills/
│       └── generate-cli/        # /generate-cli skill — invoke to create a new CLI
│           ├── SKILL.md         # Skill definition (workflow, phases, checklist)
│           ├── prompts/         # Prompt fragments referenced by the skill
│           │   ├── api-discovery.md
│           │   ├── cli-design.md
│           │   ├── auth-patterns.md
│           │   └── testing.md
│           └── templates/       # Scaffolding templates
│               └── node-cli/    # Node.js/TypeScript CLI template
├── shared/                      # Shared npm packages (bundled into CLIs at build)
│   ├── auth/                    # @funnelenvy/auth
│   ├── output/                  # @funnelenvy/output
│   ├── config/                  # @funnelenvy/config
│   └── rate-limit/              # @funnelenvy/rate-limit
└── clis/                        # CLI repos (gitignored, each has own git repo)
    ├── ga4-cli/
    ├── ahrefs-cli/
    ├── meta-ads-cli/
    ├── mailchimp-cli/
    └── buffer-cli/
```

Each CLI lives in the `clis/` directory as its own standalone git repo (gitignored by the meta-repo):
```
marketing_clis/
└── clis/
    ├── ga4-cli/           # standalone git repo
    ├── ahrefs-cli/        # standalone git repo
    ├── meta-ads-cli/      # standalone git repo
    ├── mailchimp-cli/     # standalone git repo
    └── buffer-cli/        # standalone git repo
```

## GitHub Repository Structure

### GitHub Organization
All repos live under a single GitHub org or user account. The meta-repo README is the "front door" for the project.

### Meta-Repo: `marketing-clis`
- GitHub repo name: `marketing-clis`
- Contains generator, shared libs, templates, registry
- README explains the project vision, lists all CLIs with install commands and status badges

### Individual CLI Repos
Each CLI is a **fully standalone** GitHub repo:
- Repo name: `{tool}-cli` (e.g., `ga4-cli`, `ahrefs-cli`, `meta-ads-cli`)
- Fully self-contained — anyone can clone and use without the meta-repo
- Shared packages are copied/bundled at build time, not external deps at runtime
- README links back to meta-repo: "Part of [Marketing CLIs](https://github.com/FunnelEnvy/marketing-clis) — open source CLIs for marketing tools."

### Git Init for Each Repo

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FunnelEnvy/marketing-clis](https://github.com/FunnelEnvy/marketing-clis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
