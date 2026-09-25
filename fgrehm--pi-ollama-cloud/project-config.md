---
trigger: always_on
description: Canonical instructions for local coding agents (Pi, Copilot coding agent). Claude Code does not read this file directly; add a `CLAUDE.md` importing `@AGENTS.md` if you want it to.
---

# Development Rules

Canonical instructions for local coding agents (Pi, Copilot coding agent). Claude Code does not read this file directly; add a `CLAUDE.md` importing `@AGENTS.md` if you want it to.

## Conversational Style

- Keep answers short and concise.
- No emojis in commits, issues, PR comments, or code.
- No fluff or cheerful filler text.
- Technical prose only, be direct.
- When the user asks a question, answer it first before making edits or running implementation commands.
- When responding to user feedback or an analysis, explicitly say whether you agree or disagree before saying what you changed.

## Code Quality

- Read files in full before wide-ranging changes, before editing files you have not fully inspected, and when asked to investigate or audit. Do not rely on search snippets for broad changes.
- No `any` unless absolutely necessary.
- Inline single-line helpers that have only one call site.
- Check `node_modules` for external API types; do not guess.
- No inline imports (`await import()`, `import("pkg").Type`, dynamic type imports). Top-level imports only.
- Always ask before removing functionality or code that appears intentional.
- Do not preserve backward compatibility unless the user asks for it.
- Strict TypeScript: no `enum`, no `namespace`/`module`, no `import =`, no `export =`, no parameter properties. Use explicit fields with constructor assignments if needed.

## Module Boundaries

This is a Pi extension. The split is intentional, do not collapse it:

- `index.ts` — Extension entry point. Provider registration, command registration, session lifecycle hooks. Should not call the network directly; delegate to `models.ts` and `web-tools.ts`.
- `models.ts` — Provider data plane: API constants, model assembly, fetch helpers, and the `refreshOllamaCatalog` callback wired as `refreshModels` in `index.ts`. Owns `OLLAMA_BASE`.
- `web-tools.ts` — `ollama_web_search` and `ollama_web_fetch` tool registrations. Self-contained; depends on `models.ts` for `OLLAMA_BASE` and `utils.ts` for `fetchJsonWithTimeout`.
- `config.ts` — JSON config file loader, env-var resolution, and schema validation (`CONFIG_SCHEMA` / `sanitizeConfig`).
- `thinking-levels.ts` — Per-model reasoning-effort maps. Pure data + a single resolve function reading `reasoning.generated.ts`.
- `utils.ts` — Cross-cutting helpers (`fetchJsonWithTimeout`, `concurrentMap`, `getContextLength`). Keep small.
- `scripts/generate-models.ts` — Generator script. Runs against the live API to refresh `models.generated.ts`. Not shipped at runtime.
- `scripts/generate-pricing.ts` — Generator script. Fetches the model pricing table from `https://ollama.com/pricing` plus the Ollama model list and writes `pricing.generated.ts` by matching catalog IDs to pricing rows. Not shipped at runtime.
- `scripts/generate-reasoning.ts` — Generator script. Fetches the `ollama-cloud` provider's `reasoning_options` from `https://models.dev/api.json` and writes `reasoning.generated.ts` (the reasoning-level data source; the same models.dev data pi uses for built-in providers). Runs before `generate-models` in the `npm run generate-models` chain. Not shipped at runtime.
- `scripts/generate-limits.ts` — Generator script. Probes per-model max output tokens against the live chat completions API (needs `OLLAMA_API_KEY`) and writes `limits.generated.ts`. Must be rerun whenever the model catalog changes so new models get probed limits — run it BEFORE `npm run generate-models` when new models appear, because `generate-models` bakes `maxTokens` from `limits.generated.ts` into the catalog. Not shipped at runtime.
- `models.generated.ts` — Generated output of `scripts/generate-models.ts`. **Do not edit by hand.** Regenerate via `npm run generate-models` and commit the result.
- `pricing.generated.ts` — Generated output of `scripts/generate-pricing.ts`. Shipped at runtime (in `package.json` `files`). **Do not edit by hand.**
- `reasoning.generated.ts` — Generated output of `scripts/generate-reasoning.ts`. Shipped at runtime (in `package.json` `files`). **Do not edit by hand.** Regenerate via `npm run generate-reasoning` (or as part of `npm run generate-models`) and commit the result.
- `limits.generated.ts` — Generated output of `scripts/generate-limits.ts`. Shipped at runtime (in `package.json` `files`). **Do not edit by hand.** Regenerate via `OLLAMA_API_KEY=<key> npm run generate-limits` and commit the result. Models without a probed limit fall back to 32768 in `models.ts`.

When adding a new **runtime** module (a `.ts` file that ships):

- Add it to `package.json` `files` in the same PR. CI does not catch missing entries; npm will silently ship a broken package. Test files and `scripts/` are not shipped and must not be added.

## Error Handling

- Never silently swallow rejections from `Promise.allSettled`, `Promise.all`, or `concurrentMap`. Count failures, surface them in the final notification, and let the caller decide.
- `try/finally` is mandatory for every `setTimeout` and `AbortController`. Cleanup must run on the error path, not only the happy path.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fgrehm/pi-ollama-cloud](https://github.com/fgrehm/pi-ollama-cloud) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
