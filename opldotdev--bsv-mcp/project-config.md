---
trigger: always_on
description: Contributor guidance for the BSV MCP repository.
---

# CLAUDE.md

Contributor guidance for the BSV MCP repository.

## Project overview

BSV MCP is an open-source Model Context Protocol server for Bitcoin SV. It
contains the published MCP package and the Next.js site that documents and hosts
the remote MCP endpoint.

## Current product decisions — September 8, 2026

These decisions supersede older OAuth and architecture proposals.

- Local stdio MCP is the default and active product. It requires no Sigma sign-in.
- Keep existing MCP HTTP transport opt-in and leave the deployed endpoint unchanged. Do not retire either as part of local installation or documentation work.
- Hosted MCP, its authorization server, and HTTP spending approval are deferred.
  Existing HTTP code is legacy implementation, not an instruction to finish it.
- Sigma Connect is optional and requires explicit user consent. It is separate
  from MCP authentication and from the wallet. Do not change Sigma Auth or its
  database to make BSV MCP work.
- A future hosted authorization server would belong to bsvmcp.com itself.
  This is a deferred direction, not an active implementation task.
- Local browser setup, an external wallet's signing API, and 1Sat service calls
  may use HTTP. They do not make BSV MCP a remotely hosted MCP product.
- The latest user decisions govern. Internal WORKLIST and the recent Sigma
  handoff are supporting references; old checkboxes are not authorization.

The website and legacy HTTP routes still contain contradictions with these
choices. Correct them through the current reconciliation plan; do not treat
passing tests or deployed behavior as proof of the intended product scope.

The retired Cloudflare worker and OpenNext configuration were removed. Do not
reintroduce a Cloudflare deployment path unless the product owner explicitly
requests one.

## Essential commands

```sh
bun install
bun test
bun run lint
bun run build:all       # Vite MCP app, then the published server bundle
bun run build:next      # Next.js site
bun run dev             # Next.js site locally
bun dist/index.js --stdio
```

The package version is defined in `package.json`. The published package includes
the built `dist/` bundle, README, changelog, license, and Smithery manifest.

For a local MCP connection:

```sh
bunx bsv-mcp@latest --stdio
```

The server must write only protocol data to stdout in stdio mode. Keep the
stdio guard as the first loaded module and send diagnostics to stderr.

## Repository layout

- `index.ts`, `server.ts`: package entrypoint and MCP server factory.
- `tools/`: BSV, wallet, ordinals, BAP, BSocial, MNEE, x402, and utility tools.
- `utils/`: key management, wallet initialization, backend configuration, signing,
  redaction, spending approval, and protocol helpers.
- `app/`, `components/`, `lib/`: Next.js site, hosted MCP route, OAuth
  protected-resource metadata, documentation, and landing page.
- `prompts/`, `resources/`: MCP prompts and BRC/protocol resources.
- `src/views/`: the Vite-built MCP App dashboard bundled into `dist/app.html`.
- `scripts/`: package build and tool-manifest generation.
- `docs/`: focused technical documentation. The website is the human-facing
  documentation source.
- `skills/bsv-mcp/SKILL.md`: the package skill used by supported clients.

## Authentication and transport boundary

The supported local stdio flow is: AI client → local BSV MCP → an unlocked
local Vault wallet or an explicitly connected external BRC-100 wallet. No
OAuth account is required to start or use that local connection.

A web page or loopback signing API is not a hosted MCP endpoint. Preserve
those local capabilities when isolating legacy HTTP transport code.

OAuth tokens never grant wallet authority, select signing keys, or approve a
payment. Do not expand hosted auth, modify Sigma, or add remote spending as
part of local onboarding, docs, packaging, or protocol compatibility work.

## Wallet and key safety

- Never generate keys silently during startup.
- New accounts live under `~/.bsv-mcp/accounts/<name>/` with encrypted
  `keys.bep`, `config.json`, and the network-specific wallet database.
- `BSV_MCP_ACCOUNT` selects the account and `BSV_MCP_PASSWORD` unlocks it in
  the server process. `PRIVATE_KEY_WIF` is an explicit override.
- Existing wallets should use `BRC100_WALLET_URL`; the wallet controls its keys
  and approval policy.
- Keep WIFs, mnemonics, OAuth secrets, bearer tokens, and database credentials
  out of logs, tests, fixtures, commits, and chat.
- Wallet mutations and broadcasts must respect `DISABLE_BROADCASTING` and the
  existing spending-approval flow.
- Use the redaction helpers for errors and tool responses that may contain key
  material.
- Do not create wallet directories outside `~/.bsv-mcp` without an explicit
  design decision.

See `docs/keys.md` and `docs/external-signer.md` before changing custody or
migration code.

## Backend configuration

The default 1Sat service is `https://api.1sat.app` on mainnet and
`https://testnet.api.1sat.app` on testnet. `ONESAT_API_URL` selects a
compatible deployment. Explorer, JungleBus, ordinals, content, sponsorship,
and remote wallet storage settings are separate.

Use `utils/backends.ts` as the source of truth for backend defaults. Do not

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [opldotdev/bsv-mcp](https://github.com/opldotdev/bsv-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
