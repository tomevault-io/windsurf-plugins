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

### `.meta/` publication policy (hybrid, 2026-07-02)

Two classes of `.meta/` content, two defaults:

- **Audits, plans, and research are public by default.** They explain the code, and transparency is a feature. Before committing one, run the 30-second screen below.
- **Session exhaust is private.** Handoffs, explorations, and session-state files are gitignored here and archived in the private promo repo under `ops/`. They never need screening because they never publish. (Handoffs committed before this policy remain tracked; that's deliberate — removing them from HEAD wouldn't unpublish history.)

The screen, for the public class:

1. **Third parties by name or specific role?** ("James's co-founder", "client X asked for Y", "$VENDOR's support said Z"). If yes: generalize to a role-less description, get consent, or keep the file private.
2. **Business, financial, or client information?** Deal terms, pricing, customer lists, revenue, internal roadmap items not yet announced.
3. **Credentials or secrets, even partially redacted.** Never commit them, even with `[REDACTED]`.
4. **Tone you wouldn't want cited back in six months.** Self-deprecation is fine and often valuable; gratuitous snark about a maintainer or project isn't.
5. **Plane mismatch?** Distribution/marketing-strategy content belongs in the private promo repo, not here, regardless of sensitivity.

If any item fails the screen, stop and ask the user before committing. The default for the public class is still public — screening is a filter, not a rejection.

Decided boundaries are stated in this file as facts. Direction, planning, and
anything in-flux lives in the private sibling repo
(`../easy-notion-mcp-promo/PROJECT-MAP.md` — planes, gates, task homes,
roadmap). Dev machines have it checked out; sessions without it: ask James.
If it's written here, it's true today.

## Standing priorities

- **Top product priority: tool-surface token reduction.** The v1.0.1 HTTP `tools/list` measures 7,102 cl100k tokens (41 tools). Goal: substantially reduce the always-loaded tool-listing cost via tiered/minimal descriptions and/or dynamic toolsets, without breaking the 1.0 additive-only contract. Success = a remeasured surface benchmark. Evidence and benchmark method: `../easy-notion-mcp-promo/benchmarks/token-count/` (private planning repo — see pointer at the top of this file).
- **Docs rule — token setup URL.** All setup instructions must point users to the classic integrations page (https://www.notion.so/profile/integrations) for creating the integration token — never app.notion.com/developers. Reason (per Notion's own changelog): Developer-portal personal access tokens default to an expiration (up to 1 year); classic integration secrets do not expire. A token with a silent 12-month fuse breaks unattended deployments.
- **Claims discipline.** Before changing README or any public-facing copy that makes comparative or numeric claims (token efficiency, tool counts, comparisons to other servers), read `../easy-notion-mcp-promo/claims.md` and `../easy-notion-mcp-promo/PROJECT-MAP.md` first (private planning repo). Public claims must match verified rows there.

## Commands

```bash
npm run build       # tsc → dist/
npm test            # vitest
npm run dev         # tsc --watch

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Grey-Iris/easy-notion-mcp](https://github.com/Grey-Iris/easy-notion-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
