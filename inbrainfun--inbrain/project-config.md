---
trigger: always_on
description: This is your install + operating protocol. Claude Code reads `./CLAUDE.md` automatically.
---

# Agents working on Inbrain

This is your install + operating protocol. Claude Code reads `./CLAUDE.md` automatically.
Everyone else (Codex, Cursor, OpenClaw, Aider, Continue, or an LLM fetching via URL):
start here.

## Install (5 min)

1. Install inbrain via Bun (the canonical path):
   ```bash
   curl -fsSL https://bun.sh/install | bash
   export PATH="$HOME/.bun/bin:$PATH"
   bun install -g github:inbrain-ai/inbrain
   ```
   If `bun install -g` aborts or `inbrain doctor` reports `schema_version: 0`,
   the CLI prints a recovery hint pointing at [#218](https://github.com/inbrain-ai/inbrain/issues/218).
   Run `inbrain apply-migrations --yes` to recover, or fall back to the
   deterministic install: `git clone https://github.com/inbrain-ai/inbrain.git ~/inbrain && cd ~/inbrain && bun install && bun link`.
2. Init the brain: `inbrain init` (defaults to PGLite, zero-config). For 1000+ files or
   multi-machine sync, init suggests Postgres + pgvector via Supabase.
3. **STOP — ask the user about search mode.** `inbrain init` auto-applied a
   default but printed a 9-cell cost matrix (mode × downstream model)
   preceded by `[AGENT]` markers. You MUST relay the matrix to the operator
   and confirm their choice before continuing. Cost spread between corners
   is 25x — silent acceptance is the wrong default. See
   [`./INSTALL_FOR_AGENTS.md`](./INSTALL_FOR_AGENTS.md) Step 3.5 for the
   exact ask-the-user protocol. Same banner fires on `inbrain post-upgrade`
   for existing users (search modes were added in v0.32.3).
4. Read [`./INSTALL_FOR_AGENTS.md`](./INSTALL_FOR_AGENTS.md) for the full 9-step flow
   (API keys, identity, cron, verification).

## Read this order

1. `./AGENTS.md` (this file) — install + operating protocol.
2. [`./CLAUDE.md`](./CLAUDE.md) — orientation + resolver: architecture, cross-cutting
   invariants, the reference map, inline ship rules. It routes to on-demand detail docs:
   [`./docs/architecture/KEY_FILES.md`](./docs/architecture/KEY_FILES.md) (per-file index —
   read a file's entry before editing it), [`./docs/TESTING.md`](./docs/TESTING.md) (test
   tiers + isolation lint + E2E lifecycle), and
   [`./docs/architecture/thin-client.md`](./docs/architecture/thin-client.md) (remote-MCP seam).
3. [`./docs/architecture/brains-and-sources.md`](./docs/architecture/brains-and-sources.md)
   — the two-axis mental model (brain = which DB, source = which repo in the DB). Every
   query routes on both axes. Read before writing anything that touches brain ops.
4. [`./skills/conventions/brain-routing.md`](./skills/conventions/brain-routing.md) —
   agent-facing decision table: when to switch brain, when to switch source, how
   cross-brain federation works (latent-space only; the agent decides).
5. [`./skills/RESOLVER.md`](./skills/RESOLVER.md) — skill dispatcher. Read before any task.

## Trust boundary (critical)

Inbrain distinguishes **trusted local CLI callers** (`OperationContext.remote = false`,
set by `src/cli.ts`) from **untrusted agent-facing callers** (`remote = true`, set by
`src/mcp/server.ts`). Security-sensitive operations like `file_upload` tighten filesystem
confinement when `remote = true` and default to strict behavior when unset. If you are
writing or reviewing an operation, consult `src/core/operations.ts` for the contract.

## Common tasks

- **Configure:** [`docs/ENGINES.md`](./docs/ENGINES.md),
  [`docs/guides/live-sync.md`](./docs/guides/live-sync.md),
  [`docs/mcp/DEPLOY.md`](./docs/mcp/DEPLOY.md).
- **Debug:** [`docs/INBRAIN_VERIFY.md`](./docs/INBRAIN_VERIFY.md),
  [`docs/guides/minions-fix.md`](./docs/guides/minions-fix.md), `inbrain doctor --fix`.
- **Migrate / upgrade:** `inbrain upgrade` (binary self-update + schema migrations + post-upgrade prompts),
  [`docs/UPGRADING_DOWNSTREAM_AGENTS.md`](./docs/UPGRADING_DOWNSTREAM_AGENTS.md),
  [`skills/migrations/`](./skills/migrations/), `inbrain apply-migrations --yes` (manual schema-only).
- **Eval retrieval changes:** capture is off by default. To benchmark a
  retrieval change against real captured queries, set
  `INBRAIN_CONTRIBUTOR_MODE=1`, then `inbrain eval export --since 7d > base.ndjson`
  and `inbrain eval replay --against base.ndjson`. For public benchmark
  coverage (LongMemEval, ground-truth scoring), `inbrain eval longmemeval
  <dataset.jsonl>` (v0.28.8) runs against an isolated in-memory PGLite
  per question — your `~/.inbrain` is never opened. Full guide:
  [`docs/eval-bench.md`](./docs/eval-bench.md).
- **Drive the brain to a target health score (v0.36.4.0):** the one-command
  loop. `inbrain doctor --remediation-plan --json` previews what would be
  fixed; `inbrain doctor --remediate --yes --target-score 90 --max-usd 5`
  walks a dependency-ordered plan (sync before extract, embed after
  consolidate), re-checking score between every step, refusing to spend
  past the cost cap. Empty brains (no entity pages) or unconfigured embedding
  keys hit a `max_reachable_score` ceiling and bail with what's missing.
  Three phase handlers (synthesize / patterns / consolidate) are
  PROTECTED — only trusted local callers can submit them; MCP cannot.
  Reference: [`docs/architecture/topologies.md`](./docs/architecture/topologies.md)
  and the CHANGELOG entry for v0.36.4.0.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [inbrainfun/inbrain](https://github.com/inbrainfun/inbrain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-11 -->
