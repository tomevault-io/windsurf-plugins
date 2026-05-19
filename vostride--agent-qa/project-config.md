---
trigger: always_on
description: These instructions apply to the `agent-qa` repository. Package-level `AGENTS.md`
---

# agent-qa Agent Instructions

These instructions apply to the `agent-qa` repository. Package-level `AGENTS.md`
files under `packages/*` add narrower rules for their package and take precedence
for files inside that package.

## Branding

Use `agent-qa` wherever prose, paths, package names, config names, and product
references allow it.

Allowed forms:

- Product name in prose: `agent-qa`
- Public CLI command: `agent-qa`
- Config file: `agent-qa.config.yaml`
- Local-only override file: `agent-qa.local.yaml`
- Runtime artifact directory: `.agent-qa/`
- Scoped npm packages: `@vostride/agent-qa-*`
- Docker images: `vostride/agent-qa-*`
- MCP tools and snake_case public names: `agent_qa_*`
- Environment variables and stdout sentinels: `AGENT_QA_*`

Future changes must not introduce no-separator product branding in prose,
new file names, new package names, new MCP tool names, or new environment
variables. Prefer kebab-case `agent-qa` or snake_case `agent_qa` depending
on what the syntax allows.

<!-- branding-forbidden:start -->
Forbidden examples for new work: `AgentQA`, `AGENTQA`, `agentqa`, `agentqa_`.

Use preferred TypeScript API names such as `AgentQaConfig` and
`AgentQaConfigSchema`; do not add no-separator compatibility aliases.
<!-- branding-forbidden:end -->

## Repo Map

- `packages/` - pnpm workspace packages.
- `docker/` - release Docker images for web, Android, and hook sandboxes.
- `scripts/` - release, validation, and package staging automation.
- `skills/` - source skills that are copied into the public CLI package.

Package map:

- `@vostride/agent-qa-ids` - canonical persistent ID helpers.
- `@vostride/agent-qa-core` - runtime schemas, parser, runner, reporters,
  auth, cache, memory, analytics, hooks, and shared platform contracts.
- `@vostride/agent-qa-web` - Playwright browser adapter, DOM extraction,
  action validation, smart waits, and accessibility checks.
- `@vostride/agent-qa-android` - WebdriverIO/Appium Android adapter.
- `@vostride/agent-qa-ios` - WebdriverIO/Appium iOS adapter.
- `@vostride/agent-qa-mcp` - Model Context Protocol server, tools, schema
  references, and local HTTP/stdio transports.
- `@vostride/agent-qa-dashboard-ui` - React/Vite dashboard UI assets.
- `@vostride/agent-qa-dashboard` - local dashboard server, SQLite database,
  run queue, routes, reporter, live editor, and service ownership.
- `agent-qa` - public CLI entrypoint and packaged skills.

## Commands

Run commands from `agent-qa/` unless a task says otherwise.

- Install dependencies: `pnpm install`
- Build all packages: `pnpm build`
- Test all packages: `pnpm test`
- Typecheck all packages: `pnpm typecheck`
- Run package tests: `pnpm --filter <package-name> test`
- Run package typecheck: `pnpm --filter <package-name> typecheck`
- Validate source skills: `pnpm run validate:skills`
- Validate package publishing surface: `pnpm run validate:publish`
- Check forbidden internal namespace imports: `pnpm run lint:namespace`

Prefer focused package commands while iterating. Run the root checks when a
change crosses package boundaries, changes public contracts, or touches release
automation.

## Code Style

- Use TypeScript strict mode and the existing NodeNext ESM setup.
- Follow `.prettierrc`: no semicolons, single quotes, trailing commas, and
  print width 100.
- Preserve existing module boundaries and public exports. Avoid new
  cross-package imports unless the package already depends on the target.
- Keep generated artifacts out of source changes unless the plan explicitly
  asks for them.
- Prefer existing schema, parser, reporter, auth, memory, analytics, and
  dashboard patterns over new abstractions.
- When editing YAML, configs, tests, suites, hooks, or memory files, use the
  existing schema and ID helpers instead of ad hoc string edits.

## Testing

- Add or update focused tests for the package you change.
- For CLI changes, run `pnpm --filter agent-qa test`.
- For core runtime/schema changes, run `pnpm --filter @vostride/agent-qa-core test`.
- For dashboard server changes, run `pnpm --filter @vostride/agent-qa-dashboard test`.
- For dashboard UI changes, run `pnpm --filter @vostride/agent-qa-dashboard-ui test`.
- For MCP changes, run `pnpm --filter @vostride/agent-qa-mcp test`.
- Before release-facing changes are complete, run `pnpm typecheck`,
  `pnpm run validate:skills`, and `pnpm run validate:publish`.

## Security

- Never commit `.env*`, `agent-qa.local.yaml`, auth stores, real PostHog keys,
  subscription tokens, or local credential material.
- Do not commit `.agent-qa/`, `dist/`, `.turbo/`, `.pnpm-store/`, or
  `node_modules/`.
- Do not log LLM API keys, bearer tokens, subscription-auth tokens, or
  redacted secret material in plain text.
- Keep analytics project keys on the existing release-time path. Source
  checkouts should keep analytics key material empty unless a release script is
  explicitly writing it.
- Use existing workspace/path validation helpers when accepting user-provided
  paths.
- Keep dashboard and MCP endpoints on loopback by default unless a task
  explicitly changes networking behavior.

## Package Notes

- Public packages keep `private: false`, Node `>=24`, FSL license metadata,
  npm publish config, and explicit `files` allowlists.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vostride/agent-qa](https://github.com/vostride/agent-qa) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
