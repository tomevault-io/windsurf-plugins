---
trigger: always_on
description: `@promptster/cc-audit` — point it at your local Claude Code transcripts and see where the
---

# cc-audit

`@promptster/cc-audit` — point it at your local Claude Code transcripts and see where the
money and the bad habits are. Spend attribution, model right-sizing, AI-fluency signals.
Distributed as `npx @promptster/cc-audit` (a single-file bundle) and as standalone binaries.

**Core principle: local-first, consent-tiered egress.** The deterministic half (parse →
attribute → report) runs fully local with no network and no key. Only explicit opt-in steps
send anything, each gated proportional to what leaves the machine. Preserve this — never add
a code path that phones home on a bare/non-interactive run.

## Commands

```bash
pnpm build      # tsc → dist/  (run before node dist/cli.js or the bundlers)
pnpm typecheck  # tsc --noEmit
pnpm lint       # oxlint
pnpm test       # vitest run   (pnpm test:watch for watch)
pnpm dev        # tsc --watch

pnpm build:npm  # → bundles/npm/  (esbuild single file + clean package.json; the npm artifact)
pnpm bundle     # → bundles/      (bun --compile binaries + esbuild .mjs fallback; needs bun)
```

Package manager is **pnpm**, pinned by the `packageManager` field (`pnpm-lock.yaml` is the
committed lockfile; `pnpm install --frozen-lockfile` in CI). `scripts/only-pnpm.mjs` runs on
`preinstall` and REJECTS npm/yarn — this package is not published with that guard active
elsewhere, but here it is the rule. Use `corepack pnpm` if a stale pnpm shadows the pinned
one on PATH. Always `pnpm build` before running `node dist/cli.js`.

## Architecture

`src/cli.ts` is the entry point and the only place that orchestrates I/O, prompts, and the
consent flow. Everything it calls is a pure-ish module:

- **Ingest** — `adapters/claudeCode.ts` reads `~/.claude/projects` and
  `adapters/codex.ts` reads `~/.codex/sessions` → `model.ts` `Session`/`Span` types. The
  model is tool-agnostic so a Cursor adapter can drop in later.

  **Codex is opt-in behind `--codex`, and that is a correctness choice, not caution.** The
  rails do not observe the same things: Codex ships reasoning as `encrypted_content` (no
  `thinkingChars`), has no `Read` tool (no `reads`, so no redundant-read rate) and no plan
  mode. Those fields come back 0/`[]`, which is indistinguishable from "measured and found
  absent" — averaged in silently, a Codex-heavy user's fluency signals fall because of what
  the FORMAT omits. `Session.source` exists so a signal can select its rail instead.
  Spend, tokens, models, tools and file ops are fully measured on both.

  `--codex` also writes no history snapshot, never transmits, and is REFUSED alongside
  `--judge`/`--open`: `AggregateRecord.tool` is a frozen `z.literal('claude_code')`, so a
  two-rail corpus cannot be labelled honestly at the current schema version.

  **The two rails invert the token conventions.** `TurnUsage` is ADDITIVE (Anthropic:
  `input` excludes the cache buckets). Codex is SUBSET (`input_tokens` is the total,
  `cached_input_tokens`/`cache_write_input_tokens` are inside it), so `toTurnUsage`
  subtracts them back out. Reading Codex's `input_tokens` straight through double-counts
  the largest bucket in the file — cacheRead is 303M of 313M tokens on the local corpus.
  Its other trap is `token_count`: use `last_token_usage` (per request), never a difference
  of `total_token_usage`, and suppress a row that exactly repeats the previous one. That
  suppression is what makes all 25 local rollouts reconcile to the token.
- **Analyze (local, deterministic)** — `attribute.ts` (spend by model/command), `pricing.ts` +
  `vendor/` (cost tables), `fluency.ts` / `alwaysOn.ts` / `conditionalContext.ts` (fluency
  signals), `audit.ts` (ties it together into an `AuditResult`), `aggregate.ts` (the
  privacy-safe record). `report.ts` + `theme.ts` render the TUI.
- **Egress (opt-in)** — `footprint.ts` builds task gists; `judgeClient.ts` (`--judge`
  right-sizing), `open.ts` + the `--open` upload (shareable report), `fixClient.ts` / `fix.ts`
  (`cc-audit fix` reviewable patches). All hit backend HTTP endpoints behind `CC_AUDIT_API`.
- **`index.ts`** — the importable library surface (CLI lives in `cli.ts`, not exported there).

### The bare interactive run asks TWO questions
Both default Yes, both at the very bottom, after the whole report. **Order is load-bearing**:
the analysis runs first because its output is what makes the link worth creating, and the
link's disclosure has to name what the analysis actually produced.
1. **Run the analysis now, and install the skill** (`agentRun.ts` + `skill.ts`). One yes does
   both because they cover different moments:
   - **Shell-out** (`claude -p` / `codex exec`) answers "right now" — plans printed in the
     same terminal, same run. This is the first-run experience; the skill path alone (install
     → restart session → recall a phrase) is too much friction before any value lands.
   - **Skill** is the durable path and produces *better* plans, because it runs inside a
     session with their repo loaded and can cite the actual line in the actual CLAUDE.md.
     Installing is a file write, so it rides along free.

   Both instruction sets are EMBEDDED in the binary, never fetched — they execute with the
   user's agent's permissions, so a network delivery path would be an instruction supply

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pa-arth/cc-audit](https://github.com/pa-arth/cc-audit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
