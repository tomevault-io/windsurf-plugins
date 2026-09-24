---
trigger: always_on
description: LoreKit is a Supabase-backed MCP server for shared, persistent agent memory.
---

# LoreKit — Agent Context

LoreKit is a Supabase-backed MCP server for shared, persistent agent memory.
Agents read and write *lore* (lessons) via MCP tool calls. A Next.js dashboard
lets humans browse, search, and manage those lessons.

→ For architecture, MCP tools, scope format, tokens, OTel, and deployment:
  **read [docs/](./docs/README.md) on demand — do NOT load all docs upfront.**

---

## Package map

| Package | Path | Role |
|---------|------|------|
| `@lorekit/core` | `packages/mcp-core/` | Scope validator, DB client, 10 tool handlers, OTel tracer/meter |
| `@lorekit/feature-flags` | `packages/feature-flags/` | OpenFeature-standard flag evaluation. `registry.ts` is the single hand-authored source of truth (zod-validated, all four OpenFeature value types); `nx run feature-flags:generate` projects it into typed TS bindings + a language-neutral `flags.manifest.json`. `LoreKitFlagProvider` resolves: session override → A/B(/n) experiment (deterministic FNV-1a bucketing on a stable `targetingKey`) → static default. Telemetry is two mechanisms: `featureFlagOtelHook` stamps `feature_flag.*` span attributes per evaluation (server-side), while `packages/web`'s `FeatureFlagsProvider` tags RUM signals with `feature_flag.<key>` (browser-side). UI-affecting experiments use a copy-and-suffix component convention (a resolver + one whole component per variant, never inline branches). `/settings/developer` (override UI) is open to any signed-in user outside production; in production it's gated by a server-side email allowlist (`developer-users.ts`, `notFound()` on the page). Package rules + file map: [`packages/feature-flags/CLAUDE.md`](./packages/feature-flags/CLAUDE.md). Full guide: [`docs/feature-flags.md`](./docs/feature-flags.md). Add/update/remove a flag via the `feature-flags` skill |
| `@lorekit/web` | `packages/web/` | Next.js 15 dashboard (Vercel) |
| `@lorekit/cli` | `packages/cli/` | Zero-dep Node CLI. **Setup:** `install`/`uninstall`/`doctor`/`update` (scaffold skills + MCP + lifecycle hooks into `.claude`; connectivity/token/scope health checks; offline skill refresh). **Reads:** `list`/`search`/`show`/`stats`/`scopes`/`diff`/`tree`/`lint`/`dedupe`/`link` (Offline + Remote split, `--json`/`--scope`). **Recurrence tooling:** `obligations` (a changed-file set vs a declarative Surface-Partner Map; per-entry `state` `advisory`/`gating`/`retired`) + `invariants candidates` (candidate scan reusing `dedupe`'s clustering; never auto-compiles or gates). **Maintenance:** `purge`/`purge-expired` (remote-only, account-wide, irreversible, confirm-or-`--yes`). Plus `hook` (shared hook engine behind the plugins), `mcp` (local stdio MCP server), `migrate`, `completion`. Self-contained OTLP telemetry (`service.name=cli`). Full command reference: [`docs/cli.md`](./docs/cli.md) |

| `plugins/` | `plugins/` | Per-framework deterministic bundles: `lorekit-claude` (marketplace plugin: skill + hooks + MCP), `lorekit-cursor` (rule + `stop` hook), `lorekit-codex` (feature-flagged hooks + `AGENTS.md` fallback, experimental). Root `.claude-plugin/marketplace.json` lists the Claude plugin. |
| `supabase` | `supabase/` | Edge Functions (production MCP server), migrations, NX targets |
| `@lorekit/smoke-tests` | `packages/smoke-tests/` | Live-endpoint integration/smoke tests against the deployed Edge Functions (memories, orgs, MCP, BYOD) — no application code, self-skips when its env vars are absent |

The **production MCP server** is `supabase/functions/mcp/index.ts` (Deno, self-contained). There
is no other MCP server implementation — a prior Node.js/Fly.io variant (`packages/mcp-server/`)
was never deployed and has been removed.

**Shared hook engine:** `lorekit hook --adapter <claude|cursor|codex> --event <name>` reads the host's
JSON on stdin and injects lessons / a retrospective nudge on stdout, always exiting 0. Logic lives once
in `packages/cli/src/{core,adapters}/`; each adapter reshapes I/O to its host. On a tool failure it
additionally does a best-effort lesson lookup (`failureQuery` distils terms from the tool name + error
text → `relevantLessonsFromStore` QUERIES the store — a SINGLE `store.search` carrying ALL the terms in
one call (OR semantics) across the scope hierarchy, so the offline store is walked once, not once per
term → the pure `dedupeRelevant` de-dupes the hits by `scope::key` and caps at 3, keeping the store's own
ordering) and injects any relevant prior lessons BEFORE the write-nudge — an unusable store, a throwing
search, or no match silently falls back to the nudge alone, and any error is swallowed (exit 0). This
deliberately QUERIES rather than post-filtering the SessionStart-injected set: post-filtering could only
ever resurface an already-shown lesson, so a paraphrased match or one past the per-scope read cap was
unreachable. Matching is the store's job — server-side FTS (with stemming) for
remote, full-scope substring for local — but ORDERING is not relevance: the remote handler orders by
`updated_at desc` (`supabase/functions/memories/handlers/search.ts`), and the local two-tier store puts
project-tier hits ahead of home-tier ones, so scope precedence holds only within a tier;
`store.search`'s `q` accepts a term LIST for exactly this

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mthines/lorekit](https://github.com/mthines/lorekit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
