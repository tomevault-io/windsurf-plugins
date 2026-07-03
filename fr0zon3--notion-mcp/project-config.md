---
trigger: always_on
description: Markdown-first Notion MCP server. Agents write markdown, the server converts it to Notion's block API. Agents never touch Notion block objects directly.
---

# easy-notion-mcp

Markdown-first Notion MCP server. Agents write markdown, the server converts it to Notion's block API. Agents never touch Notion block objects directly.

## Open source context

**This is open source software** (MIT-licensed, published to npm as `easy-notion-mcp` and on GitHub as `Grey-Iris/easy-notion-mcp`). Commits, PR bodies, issue responses, and diffs become part of the public record — they are cited back at the project, not just read. That should shape how you work on this repo:

- **Security claims are load-bearing.** If CI allowlists a CVE as "not exploitable" or a commit claims "we don't use the vulnerable code path," that claim needs to be grounded in actual file:line evidence from the code paths involved, not in reasoning from what we import. When in doubt, patch rather than whitelist — users install this server and hand it their Notion workspace token, so "theoretically safe" is not the bar.
- **Upstream-first for transitive fixes.** If a root cause lives in a dependency (e.g. `@modelcontextprotocol/sdk` pulling a vulnerable `hono`), file an issue or PR upstream alongside any local patch. Local overrides are a short-term workaround; upstream fixes help the whole ecosystem and reduce our long-term exception list.
- **Downstream consumers exist.** People install via `npx easy-notion-mcp`, wrap us in other MCP clients, or depend on us as a library. Consider how changes to `package.json`, `exports`, `bin`, default behavior, and tool schemas affect them — not just our own CI.
- **PR scope discipline.** Keep PRs narrowly scoped so reviewers and future auditors can tell what changed and why. Don't bundle unrelated fixes into a docs PR, don't let chores leak into feature branches. Semantic titles, focused diffs.
- **Honest positioning.** Avoid marketing superlatives in README/docs. Soften unverifiable comparisons, cite real numbers, and match the existing measured tone.

### Screening `.meta/` files before commit

Handoffs, audits, and plans under `.meta/` are public by default. That's the right default — transparency is a feature, and agent portability requires these files to travel with the repo rather than living local-only. But before committing any `.meta/handoffs/*.md` or `.meta/audits/*.md` file, run a 30-second screen:

1. **Third parties by name or specific role?** ("James's co-founder", "client X asked for Y", "$VENDOR's support said Z"). If yes: generalize to a role-less description, get consent, or move that specific file to a gitignored `.meta/handoffs-private/` path.
2. **Business, financial, or client information?** Deal terms, pricing, customer lists, revenue, internal roadmap items not yet announced.
3. **Credentials or secrets, even partially redacted.** Never commit them, even with `[REDACTED]`.
4. **Tone you wouldn't want cited back in six months.** Self-deprecation is fine and often valuable; gratuitous snark about a maintainer or project isn't.

If any item fails the screen, stop and ask the user before committing. The default is still public — screening is a filter, not a rejection. The goal is to keep the honest writing that makes handoffs valuable while catching the rare detail that shouldn't go out.

## Commands

```bash
npm run build       # tsc → dist/
npm test            # vitest
npm run dev         # tsc --watch
node dist/index.js  # stdio server (needs NOTION_TOKEN)
node dist/http.js   # HTTP server (needs OAuth creds or NOTION_TOKEN)
npm run start:http  # same as above
```

CI runs on every PR and push to `main`/`dev` (GitHub Actions: build, typecheck, test on Node 20 + 22). The package requires Node >=20 (`engines`).

## Releasing

CI Trusted Publishing is the primary path. Pushing a `v*` tag triggers `.github/workflows/release.yml`, which runs build/typecheck/test, publishes to npm via OIDC (no `NPM_TOKEN` required), and creates the GitHub release. Total runtime is under a minute when green.

1. Bump `version` in **all three** of `package.json`, `package-lock.json` (top-level `version` field AND `packages."".version`), and `server.json` (the MCP Registry manifest; bump BOTH its top-level `version` field AND `packages[0].version`) in the same commit (a CI drift-guard test, `tests/server-version.test.ts`, fails the build if `server.json` and `package.json` versions diverge, so all three files must move together)
2. Commit: `git commit -am "chore: bump to vX.Y.Z"`
3. Tag: `git tag vX.Y.Z`
4. Push: `git push public dev && git push public --tags`
5. Watch the Release workflow run (`gh run watch` or the Actions tab). It should succeed end-to-end: `npm publish --provenance` via Trusted Publishing, then `gh release create` from inside the workflow.
6. Verify npm picked up the new version: `curl -s https://registry.npmjs.org/easy-notion-mcp/latest | jq -r .version` (avoid `npm view`, which can return stale local-cached data for several minutes).
7. Merge `main` ← release PR, then `dev` ← `main` catch-up merge.

### Manual fallback

Use the manual path when CI is unavailable, the npm registry is rejecting OIDC, or you deliberately want to publish from local state. Steps 1-4 above are unchanged; replace step 5 with:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FR0ZON3/notion-mcp](https://github.com/FR0ZON3/notion-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-03 -->
