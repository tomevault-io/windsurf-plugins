---
trigger: always_on
description: - Purpose: TypeScript CLI for managing Ghost CMS instances.
---

# AGENTS.md

## Project

- Name: `ghst`
- Purpose: TypeScript CLI for managing Ghost CMS instances.
- Status: Active TypeScript CLI with tests and fixture-backed Ghost Admin API mocks covering the current command surface (`auth`, `comment`, `post`, `page`, `tag`, `member`, `newsletter`, `tier`, `offer`, `label`, `webhook`, `user`, `image`, `theme`, `site`, `socialweb`, `stats`, `setting`, `migrate`, `config`, `api`, `mcp`, `completion`).
- Documentation split:
  - `README.md`: install + usage only (end-user docs)
  - `CONTRIBUTING.md`: cloning, local development, testing, and contribution workflow

## Runtime And Tooling

- Node: `20.x`, `22.x`, `24.x` (`.nvmrc` defaults to `24`; `package.json` engines allow all three)
- Package manager: `pnpm 10.x`
- Language: TypeScript (ESM)
- Build: `tsup`
- Test: `vitest`
- Lint/format: Biome

## Quick Start

```bash
nvm use
corepack enable
pnpm install
pnpm lint
pnpm typecheck
pnpm test
pnpm build
```

## Common Commands

- Dev CLI: `pnpm dev --help`
- CLI help: `node dist/index.js --help`
- Build: `pnpm build`
- Typecheck: `pnpm typecheck`
- Test: `pnpm test`
- Lint: `pnpm lint` (Biome check: lint + formatting)
- Check committed Ghost fixtures offline: `pnpm fixtures:ghost:check`

## Documentation Rules

- Keep `README.md` focused on installing and using `ghst`; avoid repository development setup there.
- Keep all contributor/source-development instructions in `CONTRIBUTING.md`.
- Keep command/action/flag docs in sync across:
  - `README.md`
  - `AGENTS.md`
  - tests and command/runtime coverage
- When docs describe package installation UX, document intended published usage unless explicitly asked to note current publication status.

## Repository Layout

- Entrypoint: `src/index.ts`
- Commands: `src/commands/*`
- Core libs: `src/lib/*`
- Validation schemas: `src/schemas/*`
- Tests: `tests/*`
- CI workflows: `.github/workflows/*`

## Implemented Commands

- `ghst auth login|status|list|switch|logout|link|token`
- `ghst comment list|get|thread|replies|likes|reports|hide|show|delete`
- `ghst post list|get|create|update|delete|publish|schedule|unschedule|copy|bulk`
- `ghst page list|get|create|update|delete|copy|bulk`
- `ghst tag list|get|create|update|delete|bulk`
- `ghst member list|get|create|update|delete|import|export|bulk`
- `ghst newsletter list|get|create|update|bulk`
- `ghst tier list|get|create|update|bulk`
- `ghst offer list|get|create|update|bulk`
- `ghst label list|get|create|update|delete|bulk`
- `ghst webhook create|update|delete|events|listen`
- `ghst user list|get|me`
- `ghst image upload`
- `ghst theme list|upload|activate|validate|dev`
- `ghst site info`
- `ghst socialweb status|enable|disable|profile|profile-update|search|notes|reader|notifications|notifications-count|posts|likes|followers|following|post|thread|follow|unfollow|like|unlike|repost|derepost|delete|note|reply|blocked-accounts|blocked-domains|block|unblock|block-domain|unblock-domain|upload`
- `ghst stats overview|web|growth|posts|email|post`
- `ghst setting list|get|set`
- `ghst migrate wordpress|medium|substack|csv|json|export`
- `ghst config show|path|list|get|set`
- `ghst api [endpointPath]` (supports `--paginate`, `--include-headers`, `--field|-f`)
- `ghst mcp stdio|http`
- `ghst completion <bash|zsh|fish|powershell>` (`bash`, `zsh`, `fish`, or `powershell`)

## MCP Tool Groups

- `posts`
- `pages`
- `tags`
- `members`
- `comments`
- `site`
- `settings`
- `users`
- `api`
- `search`
- `socialweb`
- `stats`

## Behavior Notes

- `post create|update` supports `--markdown-file`, `--markdown-stdin`, `--html-raw-file`, and `--from-json`.
- `comment list` defaults to site-wide admin moderation semantics and includes replies unless `--top-level-only` is passed.
- `comment get` uses Ghost Admin's moderation read include set, and `comment thread` mirrors the Admin moderation sidebar by combining the selected comment read with the filtered thread query.
- `comment hide|show|delete` map to Ghost Admin comment status transitions (`hidden`, `published`, `deleted`).
- Destructive commands require the global `--enable-destructive-actions` flag; `--yes` only skips confirmation where confirmation is still required.
- `auth logout` requires `--enable-destructive-actions` when removing configured sites and confirmation when removing all configured sites; non-interactive all-site removal also requires `--yes`.
- `auth link` requires `--enable-destructive-actions` and confirmation before replacing an existing project link; non-interactive use requires `--yes`, and relinking updates the discovered project config within the enclosing repo.
- Interactive destructive confirmations emit `GHST_AGENT_NOTICE:` lines on stderr instructing cooperative agents to ask the user for approval before continuing.
- `post publish|schedule` supports `--newsletter`, `--email-segment`, and `--email-only`.
- `post delete` supports either `<id>` or `--filter` (requires `--enable-destructive-actions`; non-interactive delete also requires `--yes`).
- `post bulk` supports `--action` plus compatibility aliases `--update`/`--delete` and update fields including `--add-tag` and `--authors`.
- `member list --status` composes with `--filter`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TryGhost/ghst](https://github.com/TryGhost/ghst) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
