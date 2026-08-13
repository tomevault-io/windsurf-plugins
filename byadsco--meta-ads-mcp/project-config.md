---
trigger: always_on
description: Project memory for Codex (and any other AI assistant). This file is committed and shared with everyone working on the repo.
---

# meta-ads-mcp — agent guide

Project memory for Codex (and any other AI assistant). This file is committed and shared with everyone working on the repo.

## What this project is

A Model Context Protocol server that brokers Meta Ads API access for advertising agencies. Multi-tenant, OAuth-gated, with encrypted-at-rest token storage in Firestore. Deployed to Google Cloud Run.

- **Stack**: Node 20.10+, TypeScript (ESM), Express 5, vitest, Pino, Zod, Firestore. MCP SDK 1.29 (`registerTool` API + `ToolAnnotations`).
- **Entry**: [src/index.ts](src/index.ts) → [src/transport/http.ts](src/transport/http.ts).
- **Deploy**: push to `main` triggers [.github/workflows/deploy.yml](.github/workflows/deploy.yml). PRs trigger [.github/workflows/ci.yml](.github/workflows/ci.yml).
- **License**: MIT. **Repository is public on GitHub.**

## Conventions specific to this repo (v3.0.0+)

- **Tool naming**: `ads_*` prefix (no `meta_`). `ad_set` with underscore (not `adset`). Aligned with Meta's official MCP server (`mcp.facebook.com/ads`). See [docs/migration-v3.md](docs/migration-v3.md) for the full rationale.
- **Tool registration**: always `server.registerTool(name, { description, inputSchema, annotations }, handler)`. The legacy `server.tool(...)` API is removed.
- **Annotations**: every tool spreads one of the constants from [src/tools/_register.ts](src/tools/_register.ts) (`READ` / `CREATE` / `UPDATE` / `DELETE` / `TOGGLE` / `UPLOAD` / `TOKEN`) into its `annotations` field. Write tools prepend `WRITE_WARNING` to the description.
- **Adding tools**: see [docs/adding-a-tool.md](docs/adding-a-tool.md) for the full template + checklist.

## ⚠️ Pre-deploy / pre-commit safety rule

**This repo is public.** Anything pushed to GitHub is permanent and indexed by search engines and bots within seconds. Treat every commit as a one-way door.

Before **any** `git commit -m`, `git push`, `gcloud run deploy`, `docker push`, `npm publish`, or PR merge, the following must be true:

1. `npm run lint` passes.
2. `npm run typecheck` passes.
3. `npm test` passes (runs vitest).
4. `npm run build` passes (TypeScript build).
5. **No prohibited files in staging**: `.env`, `.env.*` (except `.env.example`), `*.key`, `*.pem`, `credentials.json`, `service-account*.json`, SSH private keys, `*.p12`, `*.pfx`.
6. **`gitleaks detect --staged --config .gitleaks.toml`** finds no secrets.
7. **No project-specific patterns** appear in the staged diff. Custom regex covers (full list in [.gitleaks.toml](.gitleaks.toml)):
   - `META_APP_SECRET=`, `OAUTH_SECRET=`, `OAUTH_APPROVAL_PIN=`, `SESSION_COOKIE_SECRET=`, `TOKEN_ENCRYPTION_KEY=`, `MCP_API_KEY=`
   - Meta access tokens: `EAA[A-Za-z0-9]{20,}`
   - `META_TOKENS={…EAA…}` (multi-tenant token map)
   - Google: `AIza[A-Za-z0-9_-]{35}`, `ya29\.[A-Za-z0-9_-]+`, GCP service account JSON
   - Generic: `-----BEGIN … PRIVATE KEY-----`, GitHub PATs (`gh[pousr]_`), AWS keys (`AKIA`)
8. `.gitignore` covers `.env`, `.env.local`, `*.key`, `*.pem`, `credentials.json`, `service-account*.json`, `dist/`, `node_modules/`.

**Never** use `--no-verify`, `git push -f` over a finding, or skip these checks because the change "looks small". Secrets routinely leak through README updates, log lines, debug output, and test fixtures.

### How to run the checks

If you have **Codex** with the user-scoped `pre-deploy-guard` skill installed:

- The skill activates automatically when you mention commit/push/deploy.
- A `PreToolUse` hook intercepts `git commit -m` / `git push` / `gcloud run deploy` and runs the guard. If anything fails, the command is blocked.
- For deep audits on large diffs, delegate to the `pre-deploy-guard` subagent (`Agent({ subagent_type: "pre-deploy-guard", ... })`).

To install the skill on your machine, see the bottom of this file.

If you don't have Codex, you can replicate the checks manually:

```bash
# Quick (pre-commit)
npm run lint && npm run typecheck && npm test
git diff --cached --name-only | grep -E '^\.env(\..*)?$|\.key$|\.pem$|credentials\.json$|service-account.*\.json$' && echo "PROHIBITED FILE STAGED" && exit 1
gitleaks detect --staged --redact --no-banner --config .gitleaks.toml

# Full (pre-push / pre-deploy)
npm ci && npm run lint && npm run typecheck && npm test && npm run build
gitleaks detect --redact --no-banner --config .gitleaks.toml
```

CI ([.github/workflows/ci.yml](.github/workflows/ci.yml)) runs the same checks on every PR and on push to `main`. The deploy job in [deploy.yml](.github/workflows/deploy.yml) depends on the preflight job, so a failed lint/test/build/scan blocks deployment.

## Sensitive variables this project handles

Source: [.env.example](.env.example). Each one is treated as a hard secret.

| Variable | What | Why it's catastrophic if leaked |
|---|---|---|
| `META_APP_SECRET` | Facebook App Secret | Attacker can impersonate the app and mint OAuth tokens for any user. |
| `META_ACCESS_TOKEN` | Long-lived Meta token (`EAA…`) | Direct API access on the owner's behalf. |
| `META_TOKENS` | Multi-tenant token map | Mass compromise of every advertiser. |
| `OAUTH_SECRET` | MCP OAuth JWT signing key | Forge MCP session tokens for any user. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [byadsco/meta-ads-mcp](https://github.com/byadsco/meta-ads-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
