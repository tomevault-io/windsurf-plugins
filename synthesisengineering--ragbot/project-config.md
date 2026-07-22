---
trigger: always_on
description: This is a **PUBLIC** open source repository. Be careful not to include confidential information.
---

# Claude Code Context: ragbot

## Repository: ragbot (PUBLIC)

This is a **PUBLIC** open source repository. Be careful not to include confidential information.

## Synthesis Engineering Ecosystem

Ragbot is the **conversational** reference implementation of the synthesis-engineering methodology. Sibling reference implementations cover other interaction primitives — keep them in mind when designing features so the ecosystem boundaries stay clean:

- **synthesis-console** — direct manipulation (browse and edit).
- **Ragenie** — procedural (workflow definition with autonomous execution).
- **synthesis-skills** — portable capability format consumed by every runtime and by external SKILL.md-compatible agents (Claude Code, Codex CLI, Cursor, Gemini CLI).

The family is open-ended and will grow. All implementations share the `~/.synthesis/` config home, the `ai-knowledge-*` workspace model, and a Python substrate library, and they integrate through Model Context Protocol (MCP) calls and a filesystem-as-source-of-truth contract. When in doubt about where a feature belongs: chat-led turns are Ragbot's territory; cross-runtime capabilities go into `synthesis-skills`; persistent state lives in `ai-knowledge-*` workspaces under the user's control.

## Architecture

```text
ragbot/
├── src/
│   ├── ragbot.py              # CLI entry point
│   ├── ragbot/                # Core library (chat, config, models)
│   ├── api/                   # FastAPI backend
│   ├── rag.py                 # RAG module
│   └── compiler/              # AI Knowledge Compiler
├── web/                       # React/Next.js frontend
│   ├── src/components/        # React components
│   ├── src/lib/api.ts         # API client
│   └── Dockerfile             # Frontend container
├── docker-compose.yml         # Full stack deployment
├── requirements.txt
└── engines.yaml               # LLM engine configurations (SINGLE SOURCE OF TRUTH)

~/.synthesis/                  # synthesis-engineering shared config home
├── keys.yaml                  # API keys (shared across ragbot, ragenie, etc.; never in repo)
├── ragbot.yaml                # Ragbot user preferences (default_workspace)
└── console.yaml               # Synthesis-console sources (also used by ragbot for repo discovery)
```

Legacy `~/.config/ragbot/{keys,config}.yaml` is read as a fallback when
`~/.synthesis/` is empty, so existing setups keep working.

**Running the stack:**
```bash
docker compose up -d
# Access at http://localhost:3000
```

## Data Location

Ragbot discovers AI Knowledge repositories from multiple sources. Resolution
order (when `--base-path` and `RAGBOT_BASE_PATH` are both unset, the index is
the **union** across these sources):

1. `--base-path` CLI argument or `RAGBOT_BASE_PATH` env (override mode: flat-parent only)
2. `~/.synthesis/console.yaml` — synthesis-console source list (the integration point)
3. `~/workspaces/*/ai-knowledge-*` — workspace-rooted layout glob
4. `/app/ai-knowledge` — Docker container default
5. `~/ai-knowledge` — legacy flat-parent convention

Workspace names are derived from the directory (`ai-knowledge-` prefix
stripped). Private repos (`-private` suffix or `.ai-knowledge-private-owner`
sentinel) are filtered unless `RAGBOT_OWNER_CONTEXT=1`.

Each ai-knowledge repo contains:
- **source/instructions/** - WHO: Identity/persona files
- **source/runbooks/** - HOW: Procedure guides
- **source/datasets/** - WHAT: Reference knowledge
- **compiled/** - AI-optimized output (auto-generated)

## Privacy Guidelines for This Public Repo

**This is a PUBLIC repository. Confidentiality is critical.**

- **NEVER** include client, company, or personal workspace names
- **ONLY** use generic placeholders: `personal`, `company`, `example-company`, `example-client`, `client-a`
- When in doubt, ask the user before committing

## Key Concepts

### Workspace System

- `user_workspace` config points to the user's identity workspace (e.g., "personal")
- Workspace folder names are usernames - do NOT rename to generic names
- Workspaces inherit from the user workspace

### Multi-User Design

- System supports multiple users with separate identity workspaces
- Different workspaces may come from different git repos
- User workspaces are private; some workspaces may be shared team repos

## Versioning

- Version is tracked in `VERSION` file (semantic versioning: MAJOR.MINOR.PATCH)
- **Maintain version numbers**: When making releases, increment the version appropriately:
  - PATCH (0.0.X): Bug fixes, minor changes
  - MINOR (0.X.0): New features, backwards compatible
  - MAJOR (X.0.0): Breaking changes
- Create git tags for releases: `git tag -a vX.Y.Z -m "Release vX.Y.Z"`
- Push tags: `git push origin vX.Y.Z`

## Development Notes

- Python CLI with FastAPI backend + React/Next.js frontend
- Uses LiteLLM for multi-provider LLM support
- Engines configured in `engines.yaml` (SINGLE SOURCE OF TRUTH for all model config)
- API keys stored in `~/.synthesis/keys.yaml` (shared across synthesis-engineering products)

### Agent Skills

Ragbot reads Agent Skills (directories containing `SKILL.md`) as first-class content alongside legacy runbooks.

Discovery sources, in priority order (later wins on name collision):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [synthesisengineering/ragbot](https://github.com/synthesisengineering/ragbot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
