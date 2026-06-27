---
trigger: always_on
description: **Tradeoff:** bias toward caution over speed for non-trivial work. For trivial tasks, use judgment.
---

# OpenCode Rules

**Tradeoff:** bias toward caution over speed for non-trivial work. For trivial tasks, use judgment.

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs early.**

- State assumptions explicitly before implementation when they matter.
- If requirements are ambiguous or multiple interpretations exist, ask instead of guessing.
- Prefer the built-in `question` tool over silent assumptions.
- If a simpler approach exists, say so and push back on unnecessary complexity.
- If something is unclear, stop, name the uncertainty, and clarify first.

## 2. Simplicity First

**Write the minimum code and config that solves the actual request.**

- No speculative features, abstractions, or configurability beyond the request.
- No single-use abstractions unless they clearly reduce complexity.
- No defensive handling for scenarios that cannot realistically happen.
- If the solution feels overbuilt, simplify it before handing it off.

Ask: **Would a strong senior engineer call this overcomplicated?** If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only fallout from your own edits.**

- Do not refactor unrelated code while making the requested change.
- Do not rewrite adjacent comments, formatting, or structure unless required.
- Match the existing style and patterns unless the user asks for a change.
- If you notice unrelated issues or dead code, mention them instead of fixing them opportunistically.
- Remove imports, variables, or helpers that your own changes made unused.

Every changed line should trace back to the user's request.

## 4. Goal-Driven Execution

**Turn requests into verifiable goals and avoid unverified claims.**

- For non-trivial work, make a short plan and track it with `todowrite`.
- Define how each step will be verified before calling it done.
- Prefer checks that prove the requested outcome, not vague confidence.
- Do not claim a fix is complete until the relevant validation actually passed.

Example plan shape:

```text
1. [Step] -> verify: [check]
2. [Step] -> verify: [check]
3. [Step] -> verify: [check]
```

Strong success criteria reduce churn, rewrites, and post-hoc clarification.

## 5. Search And Code Intelligence

**Choose the retrieval method based on the question instead of defaulting to one tool.**

- Use semantic or broad exploration only when the concept is unclear or naming is unknown.
- Prefer `ast-grep`/`sg` for source-code searches when the language is supported, even for known symbols, imports, and call sites.
- Use `grep`/`rg` for plain text, logs, errors, config keys, filenames, generated files, or when AST-Grep cannot express the search cleanly.
- Use LSP for definitions, references, symbols, and type-aware navigation when available.
- Use `ast-grep`/`sg` for structural code searches where syntax matters more than text, such as matching call shapes, JSX patterns, decorators, or import forms. If no local binary exists, use `npx -y -p @ast-grep/cli ast-grep`.
- Read only the files or line ranges needed to validate the current hypothesis.

## 6. Output Discipline

**Long tool output is the dominant context cost. Filter at the source, not after.**

- Pre-filter long-running bash output before it enters context: `--json` + `--jq`, `--quiet`, native `--limit` options, or a narrow search with bounded context. If you cannot predict the output size, redirect to a temp file and search that file instead of letting the raw output land in the conversation.
- For `gh`: inspect metadata first with `gh run view <id> --json jobs --jq '<filter>'`. Do not stream CI logs directly into context, including `--log-failed`, unless the failed step is already known to be small. If logs are needed, redirect them to a temp file and return only relevant snippets.
- For PR diffs and file retrieval: start with filename-level output. Do not pull patch bodies for lockfiles, generated files, bundled files, traces, or `dist/` output unless that exact patch is required.
- For large or generated files (lockfiles, build outputs, `dist/`, generated code, traces, files >1000 lines): use exact patterns, narrow paths/includes, and targeted line-range reads (`offset`/`limit`). Avoid broad `Grep` searches over these files.
- For lockfiles specifically (`package-lock.json`, `yarn.lock`, `pnpm-lock.yaml`): use `jq`, package-manager metadata commands, or narrowly scoped search to answer "is X present, at what version" — not a full read.
- When unsure how big a file is, check first (`wc -l`, `ls -lh`) before deciding whether a full read is appropriate.

## 7. Browser Work

For any task that involves driving a real browser — end-to-end testing, bug reproduction, exploring a UI, verifying rendered behavior, or scraping a page — delegate to the `browser` subagent. Do not attempt to call Playwright tools directly; they are gated off in the primary agent. The subagent returns a distilled summary; the primary agent makes any code changes that follow.

## 8. Noisy Information Gathering


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jleifeld/opencode-leifeld-profile](https://github.com/jleifeld/opencode-leifeld-profile) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-27 -->
