---
trigger: always_on
description: This repository contains Mailroom, a private Cloudflare email service, global
---

# Agent notes

This repository contains Mailroom, a private Cloudflare email service, global
CLI, TypeScript library, router skill, and stdio MCP server. The public
repository is `iannuttall/mailroom`. The public npm package is
`@iannuttall/mailroom`.

Read `PRODUCT.md` before making product-shape decisions. Read `CONTENT.md`
before writing user-facing copy, command help, MCP descriptions, or
documentation.

`CLAUDE.md` must remain a symlink to this file.

## Public package contract

The repository is a monorepo internally, but users install one package.

```txt
@iannuttall/mailroom       TypeScript API
@iannuttall/mailroom/mcp   stdio MCP server API
bin: mailroom              executable CLI
skills/mailroom            router skill
prompts                    automation prompts
config                     safe example configuration
```

Internal workspace packages are private. Do not publish or teach
`@mailroom/core`, `@mailroom/cli`, or `@mailroom/mcp`.

Node 22.19 or newer is required for the public package. The runtime Workers are
configured separately under `apps/`.

## Repository map

- `packages/core`: schemas, operation registry, API client, parsing helpers,
  routing, search documents, automation configuration, and shared types.
- `packages/cli`: the `mailroom` command, Keychain-backed auth, curated help,
  and human output.
- `packages/mcp`: a stdio MCP server exposing compact list, describe, and run
  tools.
- `apps/worker`: central Email Service handler, private API, D1, R2, AI Search,
  Telegram, and outbound mail.
- `apps/ingress`: small cross-account relay for signed inbound and outbound
  delivery.
- `prompts`: versioned Markdown instructions used by optional automation.
- `config`: non-secret automation examples.
- `migrations`: D1 migrations owned by the central Worker.
- `skills/mailroom/SKILL.md`: one router skill for discovery.
- `docs/index.md`: public documentation map and setup order.
- `docs/deploy.md`: central and cross-account Cloudflare deployment.
- `docs/configuration.md`: bindings, variables, secrets, and credential
  boundaries.
- `docs/gmail.md`: Gmail forwarding, Send As, SMTP, and Sent synchronization.
- `docs/testing.md`: phased installation acceptance tests.
- `docs/migration.md`: MX cutover and rollback.
- `docs/troubleshooting.md`: evidence-first setup diagnosis.
- `docs/agents.md`: agent and browser-assisted installation runbook.

## Documentation and onboarding rules

Keep public setup guides under `docs/` and include them in the npm package.

Read `docs/agents.md` before guiding somebody through a live installation.
Follow its authority, secret-handling, browser-control, Apps Script, and
handoff rules.

Setup documentation must:

- start from an empty account and state every prerequisite;
- separate Cloudflare account state, Mailroom state, Gmail state, and DNS;
- use placeholders instead of Ian's resource IDs, domains, or email addresses;
- include the check that proves each step worked;
- warn before MX changes, catch-alls, sends, deletes, or secret rotation;
- link to current primary Cloudflare and Google documentation;
- keep commands runnable from the repository root unless a different working
  directory is stated;
- explain rollback for any step that can interrupt mail.
- use task-based headings and direct instructions;
- omit project history, roadmap notes, product rationale, and documentation
  format decisions unless they change what the user must do.

When the shipped Apps Script changes, update its regression tests, integration
README, Gmail guide, troubleshooting guide, and any affected acceptance test.
A live Apps Script fix is incomplete until the tested repository file matches
the browser project again.

Do not teach browser agents to read Chrome profile data or cookies. Use an
explicit authenticated browser handoff, accessible controls, fresh page state,
and exact target checks. Leave consent and account ownership decisions with the
user.

`scripts/check-docs.mjs` verifies local Markdown links, documentation index
coverage, and the `CLAUDE.md` symlink. Keep it in the lint gate.

## Architecture rules

- TypeScript ESM only.
- Reusable behaviour belongs in `packages/core`.
- CLI, MCP, HTTP, and Agent wrappers stay thin.
- Register an operation once. CLI discovery, MCP discovery, and the API must
  use the same definition.
- Prefer structured schemas over parsing strings.
- Bound every input, list, body, attachment, and agent-facing output.
- Raw MIME belongs in R2. D1 stores relational state and bounded parsed text.
- Exact mailbox routes are the default. Catch-all routes must be explicit.
- The central Worker is authoritative. Ingress Workers never keep mailbox
  state.
- Bindings are preferred over Cloudflare REST calls inside Workers.
- Secrets use Wrangler secrets. Local API tokens use the macOS Keychain or an
  environment variable. Never save tokens in repository config.
- Every Promise is awaited, returned, or passed to `ctx.waitUntil()`.
- Keep request state out of module-level mutable variables.
- Use generated Wrangler binding types. Do not hand-write `Env`.

## Agent and MCP rules

The MCP server exposes only three tools:

- `mailroom_list_operations`
- `mailroom_describe_operation`
- `mailroom_run_operation`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iannuttall/mailroom](https://github.com/iannuttall/mailroom) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
