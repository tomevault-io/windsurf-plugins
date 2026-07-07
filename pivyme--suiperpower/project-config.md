---
trigger: always_on
description: Context for any AI coding agent (Codex, Cursor, generic LLM tools) working on the Suiperpower codebase itself. Mirrors `CLAUDE.md`. If you are running as Claude Code, read `CLAUDE.md`; everything else, read this file.
---

# AGENTS.md, Suiperpower

Context for any AI coding agent (Codex, Cursor, generic LLM tools) working on the Suiperpower codebase itself. Mirrors `CLAUDE.md`. If you are running as Claude Code, read `CLAUDE.md`; everything else, read this file.

## What this project is

Suiperpower is the open platform behind https://suiperpower.dev. Skills, knowledge, ecosystem catalog, and a CLI for shipping production Sui products. Distributed via curl one-liner. Built around an explicit anti-slop quality bar so projects survive past hackathons, not just win them.

Tagline: **Build something meaningful, on Sui**

The launch occasion is Sui Overflow 2026, but Suiperpower is a long-lived production tool, not a hackathon helper.

## Tech stack

- **Language**: TypeScript (strict, ESM, NodeNext)
- **Runtime**: Node.js 20+
- **Package manager**: pnpm workspaces
- **CLI**: zero runtime deps. The Convex backend owns its own package dependencies.
- **Backend**: Convex (telemetry + feedback only)
- **Website**: static setup assets today, full site is separate from the core CLI work
- **Skills**: plain markdown (Anthropic skill spec) with optional `references/` and `agents/openai.yaml`. Grok Build reads this same Anthropic format straight from `~/.grok/skills/`, so no Grok-specific generator exists
- **Knowledge base**: plain markdown
- **Ecosystem catalog**: plain JSON
- **Install**: bash script hosted at suiperpower.dev/setup.sh, npm package `@pivyme/suiperpower`

## Project structure

On-disk shape today:

```
suiperpower/
├── README.md, CLAUDE.md, AGENTS.md, CONTRIBUTING.md, LICENSE
├── package.json, pnpm-workspace.yaml
├── core/               publishable npm package "@pivyme/suiperpower"
│   ├── cli/            CLI source + cli/data/ ecosystem catalog
│   ├── skills/         SKILL_ROUTER.md + per-phase skill folders + skills/data/
│   ├── scripts/        release tooling (preamble injector, lint, package)
│   ├── install.sh      curl one-liner bootstrap
│   └── skills-lock.json
├── convex/             telemetry + feedback backend
└── web/                static setup assets + website shell
```

Skills live under `core/skills/<phase>/<name>/SKILL.md`. Phases: `learn/`, `idea/`, `build/`, `ship/`, `grow/`. As of today `learn/`, `idea/`, `build/`, `ship/` exist; `grow/` is planned.

## Build commands

Real today:

```bash
pnpm install                  # install workspace deps
pnpm dev                      # run CLI locally via tsx (core/cli/index.ts)
pnpm build                    # tsc to core/dist/ + chmod +x dist/cli/index.js
pnpm typecheck                # tsc --noEmit
pnpm preamble:check           # verify the telemetry preamble in every SKILL.md
pnpm package:skills           # build per-skill tarballs and index.json under web/public/skills/
pnpm lint:skills              # validate every core/skills/**/SKILL.md (frontmatter, voice, banned words)
pnpm lint:catalog             # validate every core/cli/data/*.json (schema, sort order, reachable URLs)
pnpm test                     # typecheck + lint:skills + lint:catalog + preamble:check
pnpm test:install             # CLI smoke test (build, version, doctor, vendor-mode init)
pnpm setup                    # run ./setup local-dev convenience
```

## Conventions

- ESM only (`.js` extensions in imports under NodeNext).
- Strict TypeScript, no implicit any.
- Single source of truth for branding strings: `core/cli/branding.ts`.
- Skills are plain markdown, no code generation in skills.
- Catalog data is JSON, sorted alphabetically by id.
- Naming: kebab-case for skills, files, folders, catalog ids.
- No emojis in product copy.
- No em-dashes anywhere. Use commas or periods.
- No banned words: "leverage", "cutting-edge", "world-class", "revolutionary", "AI-powered", "Web3".
- Capitalize Sui-specific terms: Move, Object, PTB, Walrus, DeepBook, Scallop, Kiosk, zkLogin.

## Quality bar

When implementing a skill or adding catalog content, hold to:

- The anti-slop quality bar this project embodies: every build / ship skill must end with a real "will this survive past the hackathon" gate, not a checkbox
- The senior-friend voice: direct, no marketing-speak, no em-dashes, banned words enforced by `pnpm lint:skills`
- The Anthropic skill format: valid frontmatter, byte-identical telemetry preamble, kebab-case naming, folder name matches frontmatter `name:`
- Mechanical doc rules: code blocks language-tagged, dates `YYYY-MM-DD`, Sui terms capitalized (Move, Object, PTB, Walrus, DeepBook, Scallop, Kiosk, zkLogin)

Every build / ship skill ends with a non-trivial Quality gate (anti-slop). Use existing skills under `core/skills/build/` and `core/skills/ship/` as canonical examples.

## Skill authoring (must read before touching skills/)

CLAUDE.md has the full skill authoring section. The short version:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pivyme/suiperpower](https://github.com/pivyme/suiperpower) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-06 -->
