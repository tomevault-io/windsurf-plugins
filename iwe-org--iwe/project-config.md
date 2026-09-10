---
trigger: always_on
description: IWE is authoritative. Choose the narrowest route; stop on success.
---

# IWE problem-solving policy

IWE is authoritative. Choose the narrowest route; stop on success.

## Non-negotiable route overrides

- For prose/summary from unknown documents, call one bounded `retrieve`. Do not precede it with metadata `find` or filesystem fallback.
- Section, both are required: `--replace '{ $header: "H", content: "<complete supplied heading block>", expect: 1 }' --delete '{ $within: "H", expect: N }'`.
- `attach` takes no `--format`; preview/apply differ by `--dry-run`.
- After a metadata-only find, compare every returned key/title with the request-derived distinctive terms before any retrieve. Generic type words do not establish relevance. If no candidate overlaps, do not retrieve merely to inspect or assess relevance. Treat that result as a terminal IWE miss and continue only with the allowed bounded fallback.
- After a terminal IWE miss on a workspace or project request, begin local recovery with one hidden-aware search for the narrowest literal field or property token; do not require related terms on one line. After one content miss, refine once or use a narrowly globbed filename, then read only the candidate source.
- After an IWE execution failure: When the request already names a source path and section or field, use one bounded direct read of only that named scope. Do not search, list, glob, or rediscover that path, heading, section, or field.
- When a requested retrieval expansion returns a seed and related documents, report the requested content for the seed and every returned document. Do not reduce the seed or an expansion to only its key or title.
- A bounded lexical find suffices for identity-only output. Stop; never verify via body reads or file search.

## Hard execution rules

- After activation, treat this file as complete IWE guidance; do not search for competing agent instructions.
- Do not use web search, `grep`, `rg`, `find`, recursive lists, or broad reads before trying IWE. The failed-search fallback below is the only workspace-search exception. Do not run routine preflight.
- Do not install, update, configure, or repair IWE.
- Missing destructive scope is a blocking input, not a discovery task: when the target set or user-owned selection criterion is undefined, refuse without tools; never inspect the workspace to invent that criterion.
- Use `iwe <command> --help` only after an IWE CLI command fails and its error does not provide enough information for one direct correction. Never call help proactively, globally, or after a successful command.
- Do not run discovery or validation as preflight before a direct operation when its target, inputs, and guards are known from the request or prior evidence. Required mutation preview is execution, not preflight validation.
- When a supported exact-key read, preview, or mutation route is known, use it before any manual reconstruction or indirect graph query. Never treat a failed exact IWE mutation or preview as permission to edit Markdown manually; follow the bounded error policy instead.
- If an apply fails after its identical guarded preview succeeded, treat the mismatch as a consistency failure: stop and report it; do not mutate through another tool or a reconstructed command.
- If an exact mutation key, selectors, replacement content, and expected counts are supplied, call 1 must be the guarded dry-run. Do not inspect the target first.
- When discovery is necessary, make it task-shaped: include known selectors/class/heading/terms and request only the needed projection/block and limit. Do not retrieve after discovery when its shaped output already supplies the required scope.
- Default result limit: 20. Use a smaller request-derived limit.
- A stated class is a hard filter: “project note” requires `--filter '{ type: project }'`; never use untyped lexical top-1. For creation, a stated semantic class sets `type=<class>`.
- Never pass 0 as a bound; it means unlimited.
- Prefer one discovery/retrieval. Call 2 is final and only for ambiguity, one page, or failed refinement; afterward use allowed fallback/report, never a third IWE lookup. Stop after sufficient evidence. Mutation calls are separate.
- Mutation safety: resolve scope, preview, validate affected keys/counts, apply identical arguments, then verify only when success cannot prove final state. Create/new are collision-guarded exceptions: use strict validation and collision policy, never `--dry-run`.

## Route and compute parameters mentally

Use the request, conversation, and prior IWE output only; do not read sources just to choose parameters.

Routes:

Consult the "Advanced and control-plane routes" section at the end of this document only when the request explicitly needs `stats`, `stats similarity`, `squash`, `export`, `normalize`, `init`, `completions`, `docs`, or unresolved command help. All ordinary read/write routes are complete below; never consult that section for them.
Exact command help is an error-recovery route, never a basic-route prerequisite.

1. **Selector:** exact key → `--key`; typed field/entity class → `--filter`; known graph anchor → relationship flag; incomplete identity → `--fuzzy`; body concepts → `--lexical`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iwe-org/iwe](https://github.com/iwe-org/iwe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-10 -->
