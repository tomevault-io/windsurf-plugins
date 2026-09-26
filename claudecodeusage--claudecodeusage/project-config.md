---
trigger: always_on
description: Repository guidance for OpenAI Codex and other agentic contributors. A faithful
---

# AGENTS.md

Repository guidance for OpenAI Codex and other agentic contributors. A faithful
Simplified-Chinese review copy lives in [AGENTS.zh-CN.md](AGENTS.zh-CN.md).

## Product identity and scope

- Claude Code Usage is a VS Code extension that reads local Claude Code and
  Codex usage logs. Claude keeps its exact token totals, cost estimates, and
  OAuth quota; Codex Beta in v2.3.0 has provider-specific local usage and
  optimization views plus clearly labelled API-equivalent cost estimates,
  without pretending those estimates are a bill or subscription charge.
- Preserve the established product identity and Claude workflows while adding
  provider-neutral contracts. Claude and Codex dashboard presentation must use
  the same provider-aware render functions and the same CSS contract.
- Prefer token-attribution accuracy over billing precision. Keep exact totals,
  labelled estimates, and point-in-time quota observations as separate concepts.
- Keep the extension local-first, lightweight, and read-mostly. Never modify
  Claude or Codex conversation JSONL files. Treat credential handling as security-sensitive
  and preserve the existing reviewed behavior.
- Add no new runtime dependencies in v2.3.0.

## Architecture boundaries

- `src/extension.ts`: activation, commands, refresh orchestration, watcher,
  coalescing, settings changes, and diagnostic output.
- `src/dataLoader.ts`: Claude JSONL parsing primitives, validation, attribution,
  and content-analysis reducers retained for exact compatibility.
- `src/claudeIncrementalIndex.ts`: the production in-memory per-file Claude
  usage index, append-tail parser, exact global deduplication, and materialized
  dashboard aggregates. Runtime refreshes must not fall back to a full-corpus
  body read or full-record aggregation.
- `src/providers/providerTypes.ts` and provider adapters: provider-neutral token,
  coverage, confidence, and limit contracts. Do not erase provider semantics.
- `src/providers/codex/`: allowed-root discovery, schema guards, exact-request
  parsing with cumulative high-water fallback, per-file aggregate index, worker
  protocol, bounded multi-worker cold backfill, and Codex facade.
- `src/codexView.ts` / `src/codexViewComponents.ts`: Codex copy and default-provider
  contracts only; they do not own HTML, client code, or styles.
- `src/settings.ts`: the `SETTINGS` catalog and `SettingsStore`; do not scatter
  direct configuration reads.
- `src/statusBar.ts`: status-bar token/cost/quota/context presentation.
- `src/webview.ts`: the single provider-aware Claude/Codex dashboard HTML and
  client behavior. Compare never sums cost or quota across providers.
- `src/i18n.ts`: all user-facing copy for all eight UI locales.
- `src/types.ts`: shared contracts.
- Read `ARCHITECTURE.md` before changing module ownership or the data flow. If
  that change is submitted for maintainer review, also provide a faithful
  Simplified-Chinese review companion.

## Safety and privacy invariants

- Never upload prompt text, response text, raw JSONL lines, absolute paths, raw
  session IDs, credentials, or local usernames. New v2.3.0 performance
  diagnostics must not log them either; do not broaden older diagnostic output
  without an explicit privacy review.
- Codex discovery is allowlisted to `$CODEX_HOME/sessions/**/*.jsonl` and
  `$CODEX_HOME/archived_sessions/**/*.jsonl` (default `~/.codex`). Never read
  `auth.json`, SQLite databases, config secrets, keychains, browser state, or
  unknown files for Codex usage.
- Additionally, `$CODEX_HOME/session_index.jsonl` may be streamed solely to
  recover the `id` → `thread_name` mapping used for real thread titles. Absolute
  paths inside titles are masked, titles stay in memory and are never persisted,
  symlinks and non-regular files are rejected, and no other field of that file
  is read.
- Raw Codex paths/session/parent IDs may exist only in short-lived local worker
  memory. Persist machine-salted pseudonymous keys and numeric aggregates only.
- Advice and optimizer network calls remain explicit user actions and may send
  only the documented digest or text the user pasted.
- New settings default to documented, non-surprising behavior. A beta provider
  may default enabled only when its allowed local directory exists and absence
  is a no-op; other experimental/approximate features default off unless an
  approved spec explicitly says otherwise.
- Do not read secret or credential files merely to diagnose a feature. Use
  redacted metadata and fixtures.
- Do not hand-edit generated files in `out/`; edit `src/` and compile.

## Codex Beta data and performance invariants

- Codex processed tokens are `input + output`. Fresh input + output is
  `max(0, input - cached input) + output`; it is a behavior aid, not a cost or
  quota equivalent. Cached input is a subset of input and reasoning is a subset
  of output, so never add either twice.
- Attribute valid request components from `last_token_usage`; its `total_tokens`
  field is active-context size, not request usage. Suppress replay only when the
  full numeric total-plus-last signature matches the same machine-salted
  rate-limit source or the immediately preceding record. If last usage is

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ClaudeCodeUsage/ClaudeCodeUsage](https://github.com/ClaudeCodeUsage/ClaudeCodeUsage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
